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

A track is chronologically ordered set of track points. Unlike the route which contains possible waypoints, a track represents the precise history of points in space and time where you were. It is a fine-grain path that records how you travelled. For example, once you arrived at a trailhead, you might unpack your bike and turn on Strava before riding off onto the trail loop. If you were to lose GPS signal by riding under an overpass, your track would continue, but would be then recorded as at least 2 different *track segments*. A track has at least 1 track segment where each segment contains a collection of continuously recorded track points [[1]](https://en.wikipedia.org/wiki/GPS_Exchange_Format).

### How Strava Records GPX Files
It is worth noting that Strava .gpx files seem to only ever contain 1 track segment. Strava has a auto-pause feature, which stops recording data when you aren't moving. This is useful for not only conserving phone battery but ultimately representing your bike ride's moving time and average speed more accurately. According to the definition of a *track segement*, we would expect multiple segments whenever this happened in the trip. Perhaps, Strava combines these multiple segments into just 1 after uploading.

## 3. Loading GPX Data into Pandas
Since the .gpx format is essentially XML, we could use a python XML library to parse the nodes, but luckily there is a simple alternative purpose built for this called `gpxpy`. To install the package, use `pip install gpxpy` or your enviroment's relevant package manager. The reference for the package can be found [here](https://github.com/tkrajina/gpxpy), though it is recommended to use a `with` statement as a context manager when reading in the .gpx file. You can find the completed process of loading .gpx data and saving .csv data via Pandas in Notebook #0.

## 4. Planning an EDA Project
Exploratory Data Analysis sometimes feels like going into an unknown forest to explore. At some times it feels exciting, but in others, EDA can be scary--often inducing data vertigo. Although our dataset from step #3 is only 6 columns and ~8,300 rows, many others can be a hundred (or more!) columns and hundreds of thousands rows (and certainly more!). I like to imagine that my unknown forest is made up of many trees from across my various files. Each tree is as tall is as its rows and as healthy-looking as its data quality--free of missing leaves or outlier burls. The 6 columns in the dataset represent 6 unique species of trees in our data forest. Each species has its own characteristics such as data type, units, and its relationship to other species. A entangled analogy, perhaps, but stay with me! Each data forest represents an exciting adventure and it makes the arduous "Understanding" phase more enjoyable to plan with such analogies in mind.

### Data Dictionary:

In lucky cases, you are given a data dictionary. This serves as a field guide that describes the purpose of each species/column within the data forest. In our case, Strava does not publish a data dictionary for its .gpx downloads, however, each column seems is easy to understand. We'll make our own field guide:

- :evergreen_tree: `track` (*int*) = a locally unique^1^ ID for a GPX track within a ride's data. _Derived column_
- :evergreen_tree: `segment` (*int*) = a locally unique^1^ ID for a GPX segment within a GPX track. _Derived column_
- :palm_tree: `time` (*datetime*) = the naive^2^ timestamp that states when the row was recorded.
- :deciduous_tree: `elevation` (*float*) = the elevation of the user in feet.
- :deciduous_tree: `longitude` (*float*) = the longitude of the user in degrees.
- :deciduous_tree: `latitude` (*float*) = the latitude of the user in degrees

^1^the number sequence is unique, but starts with 0 for each file regardless of globally how many IDs have been produced
^2^the timestamp is not recorded with timezone information. Thus is naively is local time. I know it is the Eastern timezone, though

The first two columns are essentially trees we planted in the .csv version of the ride data. They may not be useful in the future, but we can keep them for now to see if there is a difference across many files. 

:white_check_mark: **Tip:** keep track of all explicit questions that arise when looking at the data. These will be the primary search paths you can use during EDA to explore and map your data forest.

:heavy_exclamation_mark: **Warning:** when possible, write down any assumptions you have about the data. These are implicit thoughts about your data so it may not always be easy to catch them. Validating assumptions is critical to shoring up the foundation of any work built upon them.

### Modeling _Time_ in Data:

The data Strava records is meant to model reality. Location for example is estimated using three numbers of finite precision (`elevation`, `latitude`, `longitude`). The _"when"_ of something occurring is modeled by time. We all have a general sense of what time _is_. Time is always increasing and we often count it in days, hours, minutes or seconds. How we record time numerically--and how often--depends on our needs. Let's examine these different needs through the lens of an Electric Vehicle.

**Time Scales of Interest:**
- Years: a customer needs a new car and decides to lease an electric vehicle at Company X. Marketing and Sales care about this buying cycle in years
- Months: the customer needs to know how many months left are in their vehicle's lease
- Days: a grocery store wants to know how many days on average the customer takes between shopping trips to better plan its EV charger infrastructure
- Hours: the customer wants to know how long their vehicle will take to charge at their home given their current State of Charge and charger power
- Minutes: the customer wants to know how many minutes driving to the store will take
- Seconds: the customer wants to know how many seconds it takes to get from 0 to 60 MPH before choosing to lease the vehicle
- Milliseconds: the customer and vehicle design team care how long the car takes to start and its systems to boot up
- Microseconds: past this level, the customer doesn't care since the scale is disconnected from their driving experience. The powertrain design team care about the inverter control loop delay in microseconds since a good control loop will ensure the 0 to 60 MPH eletric motor response meets customer expectations
- Nanoseconds: the power module designers for the inverter care about the "reverse recovery time" of the diodes since this contributes to thermal designs and control loop limitations
- Picoseconds: representing one trillionth of a second, this time scale is not relevant to the vehicles designers, except to research scientists working on fundamental research questions