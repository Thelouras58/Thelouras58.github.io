---
layout: post
title:  "Weather Ball"
date:   2018/01/16
excerpt: "My first Android game"
project: true
tag:
- android 
- android game
- software development
- mobile development
comments: true
---
     
## My first Android game

In the last lab project of the university course "Design and Development of Mobile Applications", we were free to make whatever application we want. In this application, we must use databases, sensors, APIs, etc. 

So we decided to make a game!

The name of the game was Weather Ball.
The player can move a ball with the use of pitch/roll/azimuth and he must avoid obstacles and catch coins.

So why the name is weather ball?
A nice feature is that the levels of the game adapt in the real-life weather conditions.

To develop this game we used a lot of technologies and android programming techniques.
Let's describe in short what we used: 

For the moving of the ball, we use the rotation matrix from the sensor manager to get the roll.

For the weather feature, we get from Geolocation the coordinates of the phone and send them to OpenWeatherApi. The open weather API returns the weather in this location.

We use 2 databases. One local SQLite on the phone and one remote firebase NoSQL.

For version control we used Github.
Adobe Photoshop for the GUI.
Audacity for the sounds.

You can read the full revportof the project [here](https://drive.google.com/open?id=1y8Mw7jFEvIboLpTVxipmXiU-XxDtsov-)(Greek).





 


