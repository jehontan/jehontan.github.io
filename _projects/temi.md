---
layout: page
title: Temi Patrol Robot
description: Repurposed COTS concierge robot
importance: 1
category: work
# related_publications: einstein1956investigations, einstein1950meaning
---

DSTA acquired a couple of low-cost concierge robots from [Temi](https://www.robotemi.com/) to experiment with directing visitors to many meeting rooms. DSTA Security was interested in using the same robots for security patrols.

<div class="d-flex flex-row justify-content-center">
    {% include figure.html path="assets/img/temi.png" class="img-fluid rounded z-depth-1" zoomable=false caption="The Temi concierge robot."%}
</div>

The Temi robot is essentially an Android tablet on a differential drive platform, and the SDK was quite limited. It was possible to manually teleoperate the robot to locations and set them as checkpoints, between which the robot can autonomously navigate. However for the security partol use-case DSTA Security also wanted live video from the robot, the ability to monitor and teleoperate multiple robots from a central control terminal.

To overcome the limitations of the SDK, I developed: a command web app; a client web app for the robot, and; an Android app that wraps the client web app and interacts with the Temi SDK. The client web app captured the video feed from the Temi tablet front-facing camera using the standard [MediaDevices API](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia). Video was transmitted to the command web app via WebRTC using the [PeerJS](https://peerjs.com/) library, where the video feeds of multiple robots were displayed ina grid allowing simultaneous monitoring. Commands (go-to-waypoint, waypoint sequence, teleoperation) were transmitted from command to client web app also via WebRTC. Lastly, the Android app on the robot exposed the required Temi SDK functions to the client web app through [JavascriptInterfaces](https://developer.android.com/develop/ui/views/layout/webapps/webview#BindingJavaScript), enabling to robot to act on the commands received.

Below is a video showing the complete system in action.

<div class="d-flex flex-row justify-content-center">
    {% include video.html path="assets/video/temi_demo.mp4" class="img-fluid rounded z-depth-1" controls=true width="400px" %}
</div>

An additional advantage of this approach is that any conformant browser could load the web app and act as client, sharing video feed from the browser's device to the command interface. Staff could load the access a URL on their device to load the web app, and use their camera to report indicents.

This project was completed in early 2020.