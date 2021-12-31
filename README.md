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

- :evergreen_tree: `track` (*int*) = a locally unique<sup>1</sup> ID for a GPX track within a ride's data. _Derived column_
- :evergreen_tree: `segment` (*int*) = a locally unique<sup>1</sup> ID for a GPX segment within a GPX track. _Derived column_
- :palm_tree: `time` (*datetime*) = the naive<sup>2</sup> timestamp that states when the row was recorded.
- :deciduous_tree: `elevation` (*float*) = the elevation of the user in feet.
- :deciduous_tree: `longitude` (*float*) = the longitude of the user in degrees.
- :deciduous_tree: `latitude` (*float*) = the latitude of the user in degrees

<sup>1</sup><sub>the number sequence is unique, but starts with 0 for each file regardless of globally how many IDs have been produced</sub>

<sup>2</sup><sub>the timestamp is not recorded with timezone information. Thus is naively is local time. I know it is the Eastern timezone, though</sub>

The first two columns are essentially trees we planted in the .csv version of the ride data. They may not be useful in the future, but we can keep them for now to see if there is a difference across many files. 

:white_check_mark: **Tip:** keep track of all explicit questions that arise when looking at the data. These will be the primary search paths you can use during EDA to explore and map your data forest.

:heavy_exclamation_mark: **Warning:** when possible, write down any assumptions you have about the data. These are implicit thoughts about your data so it may not always be easy to catch them. Validating assumptions is critical to shoring up the foundation of any work built upon them.

### Modeling _Time_ in Data:

The data Strava records is meant to model reality. Location for example is estimated using three numbers of finite precision (`elevation`, `latitude`, `longitude`). The _"when"_ of something occurring is modeled by time. How we record time is dependent on the following specifications:

- **Origin/Reference Point**: when comparing sequences of time, it is useful to refer to the same starting point. Time zones are the easiest way to reference a local time (e.g. 10:30 AM) across the globe.
- **Time Scale**: the granularity of recorded time. This is based on the needs of the analysis or design.
- **Discretization**: time is a continuous quantity, but we cannot record infinite samples so we must record a discrete point in time.
- **Synchronicity**: when you capture discrete points in time, we can do this synchronous to a sampling frequency (time series data) or asynchronously when events occur (event series data).

#### Strava's Model of Time
We can see from the first couple of rows of the `time` column, that the timestamps are recorded with date (year, month, day) and time information as granular as seconds: <br>![](images/time_glance.PNG)

The time intervals between rows seem to either be 1 second or 2 seconds. So even if our data are discrete points in time, they are not quite synchronous. This is something we will need to fix since we want to deal with a time series.

The timescale of seconds is perfectly reasonably, though. As a bike rider with human limits, any motions I make can be described with enough detail at this timescale. The "origin" or timezone for the timestamps are naive and otherwise unclear. As the person generating the data, I can say that it is created in the Eastern timezone. Since I will only be comparing my trips, these naive timestamps are fine. It may be necessary to adjust for daylight savings in the future though. The question of when "night" begins/ends is related to this information, but perhaps we can find these historical sunsets/rises with an API.

### Questions & Assumptions:

The initial questions and assumptions form the start of the data adventure. These will guide where to look in the data forest and will check risks for any danger later on. I'll start by listing my initial questions

#### Questions
1. :question: (*Data Quality*) What is the sparsity of each column?
2. :question: (*Data Quality*) Are there any outliers in each column?
3. :question: For categorical or ID columns (`track`,`segment`), how many unique values are there?
4. :question: How can we determine speed from the available columns?
5. :question: How can we summarize a ride similar to Strava?:
&nbsp;&nbsp;&nbsp;&nbsp;- Elapsed time & moving time
&nbsp;&nbsp;&nbsp;&nbsp;- Average speed
&nbsp;&nbsp;&nbsp;&nbsp;- Elapsed distance & elevation gain/fall
6. :question: Discounting stops/starts, how can we determine the "moving average speed" of a trip?
7. :question: How can we determine the "continuity" ride by factoring in stops and starts?
8. :question: How can we associate out-and-back trips recorded in different Strava uploads? Should we? 
9. :question: How can we enrich the ride's summary with weather data?
10. :question: How can we determine what percent of a ride is at night?
11. :question: How can we assign a unique trip/ride ID to each file?
12. :question: How can we determine grade/slope from `elevation` data?
13. :question: How can we ensure privacy on route data which contains home addresses?
14. :question: How can we create segments similar to Strava? How do we store them?
15. :question: How can we assess relative performance on key segments?

#### Assumptions
1. :exclamation: The units of `elevation` are in feet, not meters or some other unit.
2. :exclamation: The units of `latitude`/`longitude` are in degrees, not a "degrees,minutes,seconds" format.
3. :exclamation: The `time` column is mostly synchronous with gaps between samples no more than 2 seconds.
4. :exclamation: We can ensure privacy while maintaining integrity in the summary w.r.t distance and time.

#### Prioritizing the Places to Explore
With nearly 20 questions and assumptions to explore, it seems like a good idea to plan the starting point for our adventure and exploration. Not all questions or assumptions are equal. Some are more difficult to answer or address. For instance, enriching a trip's summary with weather data will require integration with an external API. Verifying that units of `elevation` is much easier, however.

To prioritize each question and assumption, we can assign each a relative "impact" and "feasibility". The impact of a question is related to how much insight the data can unlock if answered. For assumptions, impact is how high (or low) of a risk the assumption presents to future analysis should it be proved false later. In either case, the estimations of impact are arbitrary, but based on instinct. You can get better at gauging impact the more you explore data. The feasibility of a question or assumption is more tangible--though still relative. How much time will it take to fully investigate that item? Low feasibility items require much more time and effort to investigate. 

Items high in impact and feasibility are called "**No Brainers**". Do these first. Past a certain point if the effort to too high, or the impact too low, that item might be an "**Explore Later**". These are nice-to-haves. If the feasibility and impact are both low, then you probably don't need to address it now unless it this evaluation changes later. For a first pass at EDA, select all of the "**No Brainers**"" and a few important "**Expore Laters**". 

I've attempted to prioritize my questions and assumptions here:

_Placeholder_: 1/x curves for Impact vs. Effort and the regions of "no brainers", "do laters" and "do not dos"

### Project Directory Structure:

Staying organized both with the various data files, exploration notebooks and supporting tools is important. A repeatable project directory structure will help streamline each adventure too.

