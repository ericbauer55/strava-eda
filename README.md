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

Great, we just downloaded a .gpx file, but what exactly is it? The 