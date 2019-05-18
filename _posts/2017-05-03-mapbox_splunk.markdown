---
title:  "Enhancing Splunk Visualizations with Mapbox"
tags: MapBox Splunk GeoViz Blogs
---
Enhance the out-of-the-box visualizations provided by Splunk for cluster map visualizations by integrating with the MapBox API.
t has been possible to add custom tiles to cluster map visualizations in Splunk, but the options for adding tiles were limited because it was unclear whether external APIs integrated with Splunk.

This blog shows you how to integrate with the MapBox API to use the styles included with MapBox or custom maps.

### Before you start!
- Requirements:
  - MapBox Account & API Key
  - Internet access for your Splunk instance
  - Geographical data in the Splunk platform

- Optional:
  - Missile Map Visualization (https://splunkbase.splunk.com/app/3511/)

### Get started: Add latitude and longitude coordinates to data
- To use any geographic visualization in the Splunk platform, you need data with latitude and longitude coordinates tagged to each event. External network traffic data is a great data source that you can tag with geographical coordinates.

This example search takes Netflow events from an Intrusion Prevention System (IPS) running Suricata located on the public internet and uses the iplocation search command to create latitude and longitude fields for each event based on the src_ip and dest_ip fields. It also appends a prefix to the fields created by the iplocation command to track the flow of the data. See the Search Reference manual for more information on the iplocation command. (http://docs.splunk.com/Documentation/Splunk/6.6.0/SearchReference/Iplocation)

```
index=suricata event_type=flow
| iplocation src prefix=start_
| iplocation dest prefix=end_
```
This prefix isn't required for all cluster maps, but the Missile Map visualization expects data in this format. The exact data format expected by each visualization depends on which one you want to use. When you select a visualization, a search fragment that shows you how to format the data appears.

For example the Missile Map expects data in the following format:
![Image of Missle Map](http://tellez.sfo2.digitaloceanspaces.com/missile_map_example_search.png)

In order to format the data correctly for the Missile Map, you need to complete a few more steps.

- Make a table out of the data
- optional:
  - enable animation
  - enable pulse
  - enable different colors by application detected.


Create a table of the data with the type of application protocol detected by Suricata. In order to keep the total number of connections low, use a short-duration real-time search to make this table. This example covers the last 5 minutes and removes connections that are missing geographical information.

```
index= suricata event_type=flow
| iplocation src prefix=start_
| iplocation dest prefix=end_
| search start_Country="*" end_Country="*"
| table start_lat start_lon end_lat end_lon app
```

This search enables the Animation and Pulse options using eval:

```
| iplocation src prefix=start_
| iplocation dest prefix=end_
| search start_Country="*" end_Country="*"
| table start_lat start_lon end_lat end_lon app
| eval animate="yes", pulse_at_start="yes"
```

Using eval, create a case statement to use different colors for each type of application protocol detected. These colors come courtesy of flatuicolors.com.


```
| iplocation src prefix=start_
| iplocation dest prefix=end_
| search start_Country="*" end_Country="*"
| table start_lat start_lon end_lat end_lon app
| eval animate="yes", pulse_at_start="yes"
| eval color = case (
    match(app, "ssh"), "#c0392b",
    match(app, "dns"), "#e67e22",
    match(app, "tls"), "#f1c40f",
    match(app, "http"), "#27ae60",
    match(app, "dcerpc"), "#2980b9",
    1==1, "#7f8c8d")
```

![Table for Missile Map](http://tellez.sfo2.digitaloceanspaces.com/table_for_mapbox_missile_map.png)


At this point, you've created the custom tile that you can use for your visualization. To add the custom tileset, select the visualization tab, select missile map and select format >
![Configure Missile Map](http://tellez.sfo2.digitaloceanspaces.com/missile_map_options.png)

To update the tileset, you need two different pieces of information:

- API Token
- MapBox Style URL

##### API Token
The following help page shows you how to create a new access token for the Mapbox API. (https://www.mapbox.com/help/create-api-access-token/)

#### MapBox Style URL
There are two options for the style URL. The first option is a custom map that you or someone else has created and shared. MapBox also provides free styles for all MapBox users with a valid API token (https://www.mapbox.com/api-documentation/#styles). For this example we will use a free style provided by MapBox. The help documentation to locating the URL for your custom map can be located on the following page. (https://www.mapbox.com/help/define-style-url/)

```
mapbox://styles/mapbox/streets-v9
mapbox://styles/mapbox/outdoors-v9
mapbox://styles/mapbox/light-v9
mapbox://styles/mapbox/dark-v9
mapbox://styles/mapbox/satellite-v9
mapbox://styles/mapbox/satellite-streets-v9
```

The following code snipped from (https://www.mapbox.com/api-documentation/#retrieve-tiles) explains the syntax required to retrieve the tiles for usage with Splunk Enterprise.

```
/v4/{map_id}/{z}/{x}/{y}{@2x}.{format}

$ curl "https://api.mapbox.com/v4/mapbox.streets/1/0/0@2x.png?access_token=your-access-token"
```
The most important part of the code snippet is the format, as it gives us all of the 3 required arguments for Splunk to make the correct API request. The final syntax will look like:

`https://api.mapbox.com/v4/mapbox.streets/{z}/{x}/{y}@2x.png?access_token=your-access-token`

![Configured Missile Map](http://tellez.sfo2.digitaloceanspaces.com/missile-map_configured_tileset.png)
![Configured Missile Map](http://tellez.sfo2.digitaloceanspaces.com/mapbox-missile-map-us-result.jpg)

### Conclusion
Hopefully through this exercise you have learned how to improve your geographical visualizations with MapBox. Customers can leverage their own custom styles to personalize the tilesets and add additional context to their data.

![DefCon1](http://tellez.sfo2.digitaloceanspaces.com/animated_missile_map_defcon1.gif)
