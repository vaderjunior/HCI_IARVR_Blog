+++
date = '2025-11-13T14:03:36+01:00'
draft = false
title = 'Week 4: Thinking, thinking and more thinking'
+++

So,

This week was mostly spent on thinking and trying to work out what locomotion technique I want to use for the project.

Firstly I tried to remember and dig up all the things previous years’ students did, not just to be inspired but to do something that is different from what they have done.

Also I tried familiarising myself with the whole Unity and locomotion system in general.  
The video and playlist below were super useful. The guy is fun to listen to and overall I learned how to calibrate the controller feedback to actual input for movement and how to fine tune it. Even though everything is not clear I thought I would look up more when I actually start the project.

After a few days of planning and scrapping a ton of stuff I came to finalize the idea below for my locomotion technique.

### Air Scooter from Avatar: The Last Airbender

{{< video src="airscooter.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

I thought about how to translate this into a locomotion technique for 2 days and finally came down to the controls below.

- **Right hand:** rotate it in circles to lift the ball up. It will basically be like flappy bird where the rotation pushes the ball up but as soon as you stop it slowly starts coming down.
- **Left hand:** when it is held flat there is no movement. But if I tilt it to the front the player moves to the front, back means back, left and right as well. It is not just those 4 but combinations also work.

So the player needs to use both hands to guide Aang on top of the AirScooter to collect all the coins.

An idea of the movement is shown below.

{{< video src="locomotion2.mp4" autoplay="true" loop="true" muted="true" playsinline="true" >}}

The coming week I will start with implementation of this technique.

-Ajay
