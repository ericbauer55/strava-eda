# Strava Cycling EDA

This Python repository covers the Exploratory Data Analysis (EDA) for 3 bike rides recorded by Strava. I've chosen to analyze only 3 rides so that the differences among them can be explored without worrying about the other 250+ rides of data.

EDA will cover the following areas:

- Accessing the GPS data from Strava's website
- What a GPX is
- Loading .gpx files into a Pandas dataframe
- Common enrichments needed to enhance the usefulness of the base data
- Data quality concerns
- Ensuring data privacy when sharing with others

A brief data dictionary will be provided as well.


## 1. Accessing the Ride Data

In order to access the ride data from Strava, do the following:

1. Navigate to [Strava's website](https://www.strava.com/) and sign in to your account
2. After logging in, Strava will bring you to your "dashboard". In the upper left is a box which contains a count of your activities. This count is also a link to the list of your activities, so click on it <br> ![](images/dashboard.png)
3. Find a ride that interests you and click on it. If you want to share the data for that ride, it is recommended to choose a ride that doesn't start or end at your current residence (or any previous). 
4. Underneath the overview there is a box with an ellipsis icon (...). Click that and then hit "Export GPX" to download the data for that ride <br> ![](images/export_gpx.png)
5. Move the .gpx file to the directory you'll be working in

## 2. What is a GPX File?

Great, we just downloaded a .gpx file, but what exactly is it? GPX is short for GPS Exchange Format, but before diving into a conceptual discussion, let us take a look at the inside of a .gpx file to gain some intuition. An abbreviated version of the file is shown below with a few hierarchical annotations: <br> ![](images/gpx_file_structure.PNG)*View inside the .gpx file*

First we see that the file is broken up into an XML tree structure with a `<gpx>` node as the parent of all other data. Besides some metadata sprinkled throughout (trip start time, trip name, etc.), the file is made up of a **Track** node. A **Track** contains at least one **Track Segment** which in turn contains multiple **Track Points**. Each **Track Point** contains the chronologically ordered "story" of my bike ride. It gives us GPS coordinates (latitude and longitude) as well as the elevation and timestamp of when we were at that point. Although I do not have a health tracking device to synchronize with Strava, I imagine that other data channels like heart rate or blood-oxygen content might appear here. 

So we have an idea about how the ride data is modeled in an XML tree, but what does it physically represent? The following cartoon describes the physical activity: <br> ![](images/gpx_cartoon.PNG)*Physical activities represent by a .gpx file*

In general, a .gpx file can contain *routes* and *tracks*. A route is an unordered collection of *waypoints*--a pair of GPS coordinates at a minimum--and represents a coarse-grain path from start to destination. A route example is the GPS route plotted by Google Maps to navigate your car to a certain trailhead. There might be multiple waypoint sequences that could be routes you might take to the destination, but typically your navigation app will select the best (i.e. shortest) route for you. You can always miss a turn in which case the projected route updates to handle this change. 

A track is chronologically ordered set of track points. Unlike the route which contains possible waypoints, a track represents the precise history of points in space and time where you were. It is a fine-grain path that records how you travelled. For example, once you arrived at a trailhead, you might unpack your bike and turn on Strava before riding off onto the trail loop. If you were to lose GPS signal by riding under an overpass, your track would continue, but would be then recorded as at least 2 different *track segments*. A track has at least 1 track segment where each segment contains a collection of continuously recorded track points ^[1](https://en.wikipedia.org/wiki/GPS_Exchange_Format).

### How Strava Records GPX Files
It is worth noting that Strava .gpx files seem to only ever contain 1 track segment. Strava has a auto-pause feature, which stops recording data when you aren't moving. This is useful for not only conserving phone battery but ultimately representing your bike ride's moving time and average speed more accurately. According to the definition of a *track segement*, we would expect multiple segments whenever this happened in the trip. Perhaps, Strava combines these multiple segments into just 1 after uploading.