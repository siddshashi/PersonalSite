+++
authors = ["Lone Coder"]
title = "DroneTracker"
date = "2024-07-14"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags = [
    "hugo",
    "markdown",
    "css",
    "html",
]
categories = [
    "theme demo",
    "syntax",
]
series = ["Theme Demo"]
aliases = ["migrate-from-jekyl"]
+++

Last summer, I was invited to the AGI House AI Robotics Hackathon, an exclusive event in Silicon Valley that brought together over 500 engineers from around the world. My team and I won 2nd place for developing DroneTracker, a real-time person-tracking system using the DJI Tello's Python SDK. [GitHub][ghlink]

[ghlink]: https://github.com/AntonioMacaronio/Drone-Robotics

## Motivation

This project tackles the challenge of low-cost autonomous target tracking, a problem relevant to military reconnaissance and search-and-rescue operations. High-risk scenarios often involve soldiers or first responders navigating unfamiliar and potentially dangerous environments. DroneTracker serves as an initial step toward an alternative approach—if a low-cost drone can map and track a scene in real time, why risk human lives unnecessarily?

## Overview

Our end-to-end pipeline consists of the following steps: 

1. Real-time video input from the drone’s RGB camera
2. Pose estimation using a MobileNet-based U-Net
3. Control to adjust the drone’s position

By leveraging a lightweight deep-learning model, we achieved latency under 20ms per frame, enabling real-time tracking and responsive drone movement.

### Pose Calculation

We use MobileNet, a U-Net-based deep-learning model trained on human poses. This architecture is optimized for speed and efficiency, making it ideal for real-time tracking on edge devices. The model detects key facial and body landmarks, including eyebrows, nose, shoulders, neck, as well as positional data relative to the camera's field of view. These points inform the drone’s movement to maintain the subject at the center of the frame.

### Control

DroneTracker continuously adjusts its flight path based on the detected pose:

1. Rotation: Adjusted based on the nose position relative to the image center
2. Vertical movement: Controlled by the distance between the horizontal bisector and the nose position
3. Horizontal movement: Determined by the pixel distance between the neck and nose

This enables smooth and autonomous tracking of a moving target with minimal drift.
