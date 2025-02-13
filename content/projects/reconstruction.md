+++
authors = ["Lone Coder"]
title = "Affordable 3D Scene Reconstruction"
date = "2024-12-17"
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

With my DJI Tello Drone, I collected video footage of my bedroom interior and created a 3D reconstruction of it using Gaussian splatting. Then, analyzing the reconstruction's point cloud, I found holes in the reconstruction. From those holes, I collected additional video footage with my drone and refined my 3D reconstruction. [GitHub Link][ghlink]

## Problem 

The project addresses the challenge of creating accurate 3D reconstructions of real-world environments, specifically buildings, using accessible and cost-effective tools. Current solutions often rely on expensive equipment or complex workflows, which limit accessibility for small-scale researchers, hobbyists, or budget-constrained projects.

### Why I'm Interested 

With ongoing wars, specifically those breaking large news stories in Ukraine and Israel/Palestine, I've been interested in how engineers are building solutions to save mercilessly-lost lives. I firmly believe that as an engineer, I have power to build tools that impact lives, and the defense sector is a place in which I can to do so directly and on a large scale.  

As I understand, the most dangerous military operations today involve soldiers traversing through unknown closed corners. With this project I'm attempting to build the groundwork for a solution -- if a scene can be mapped out by a cheap drone, why would a person ever have to venture inside? 

And beyond defense, the ability to survey a scene at a high resolution while keeping cost low has huge applications in security, agriculture, architectural preservation, urban planning, AR/VR, and so much more. Imagine a personal bodyguard that flies around people who need protection. Or maybe a poaching detector surveying an African safari. Drone vision will lead the future of these spaces, and with this project I want to be a driving force in the ensuing disruption. 

### Current Solutions and their Limitations

One common approach for 3D reconstruction is LiDAR scanning. While LiDAR gives high accuracy, regardless of the lighting conditions, its main drawback that this project solves is its general-purpose accessibility. Drones with LiDAR systems cost upwards of \$15,000 while the DJI Tello drone is only \$100. 

Another common approach is traditional photogrammetry, where photos are used to build 3D models with softwares like Agisoft Metashape or Meshroom. The problem with this is that constructions are resource intensive, and the Gaussian Splatting model that I propose is much lighter weight and quicker for real-time processing. 

Finally, scenes can be reconstructed with manual modeling, which ultimately gives the highest control over the final model. However, this comes at the cost of intense labor and requires significant expertise with precise measurements. 

## Method

This project offers a low-cost, real-time solution for 3D reconstruction using a widely available DJI Tello drone and a streamlined workflow integrating Gaussian reconstruction with point cloud manipulation. Key innovations include hole detection for Gaussian splatting output and subsequent autonomous refinement of the scene based on the holes.

### Data Collection

Using the drone's SDK, I was able to program drone movements and built-in functions. Thinking about how to best capture the interior of my bedroom, I initially thought to have the drone takeoff and rotate 360 degrees, capturing video the entire flight. With this, I quickly saw two problems. First, the drone had no forward tilt capabilities, so the forward-facing camera didn't allow me to capture the entire height of the walls of my room in a single pass. Second, for the Gaussian splat to have adequate information about the objects and features of my room, it needed more information about the depth of the scene which it couldn't accurately achieve from just a simple rotation. I solved these issues by adding lateral movement through having the drone rotate in multiple circles of different heights and radius 1.5 meters. This proved to be sufficient as it was able to capture essential information for the room reconstruction.  

### Data Processing

After collecting a video from my drone, I fed the footage into Jawset Postshot to create the reconstruction and point cloud. From the footage, Postshot automatically mapped correspondences, created the point clouds, and splatted Gaussian primitives onto the scene. After doing this, as seen in Figure 2, I actually had a pretty decent reconstruction, but there were incomplete places in the scene that would have benefited from more data. 

### Hole Detection

From the Postshot output, I extracted the point cloud of the scene to perform hole detection. This started with getting a bounding box for the scene, as there were a lot of noisy data points outside of the reconstructed room. Since the points which represented my room were spatially close together, I thought to use clustering to select only those specific points. I used density-based clustering for this (DBSCAN) instead of K-means clustering because the room was of an arbitrary shape. After clustering, I removed points that weren't in any clusters from the working set. 

Next, I wanted to get meaningful coordinate locations of points on my room to later turn into waypoints for my drone, so I wanted to rotate the points such that the reconstruction walls lined up with the visualization axes. To detect where the plane was in the reconstruction coordinates, I used RANSAC and got the normal vector of the ground plane. After that, I rotated the points with simple dot and cross products such that the normal vector of the ground was aligned with the z-axis. I then continued to do the same with x-y alignment. 

After aligning, there were still some points outside of the room boundaries which I removed by only including points within the middle 95th percentile of the x, y, and z coordinates. This gave me a final filtered point cloud for hole detection. 

After that, I detected holes by iterating over all the locations within the room and recording how far they were from other points in the point cloud. To do this, I used a K-dimensional tree for fast querying which sped up operations significantly. If a location was far from any points in the point cloud, I labeled it a hole, and then with those labels, as you can see in Figure 3, I had a new point cloud just representing holes of the scene. I guess this is sort of like the negative of the original point cloud. 

Visually, I was able to see a couple of things. First, there is a large hole in the center of my room, but rightfully so -- that was the inside of my drone's flight path that I intentionally didn't bother to capture. Second, the regions needing the most refinement were in the corners of the scene, and I needed a some way to waypoints for my drone from those region locations. 

To do this, I now clustered the hole locations to label regions in the point cloud. Then, for each region I wanted to recollect footage for, I averaged the coordinate locations of all the points making up the region to get the region centroid.  

### Data Recollection

Given a centroid location, I needed a way to make my drone collect footage at and around that location. I couldn't simply route my drone to the location -- I needed it to be far away to capture footage there but close enough for that footage to be meaningful and detailed. Since my drone only has a forward-facing camera with no tilt, I chose to do the following: I routed my drone to the height of the target location and had the drone then go 2 meters radially outward from the center of my room toward the target. While this worked, it certainly could have been done more scientifically and it felt like a hack solution. Perhaps I could have used intrinsics about the camera to compute an ideal location to capture the footage, I plan on investigating that further. 

### Note on Calibration

To synchronize my drone's actual location with where I want it to be in my coordinate space, I placed a piece of tape at the center of my room by measuring the midpoints of the length and width of the room. This center served as the center of the drone paths and I was able to represent this as a coordinate in the point cloud by taking the midpoint along the walls of the room. While there was no active SLAM taking place in this project, I hope to extend this project further by implementing SLAM and remove the need to calibrate in such a way. 

### Results

As can be seen from the images of my reconstruction of a hole in the scene, I was able to create a more realistic representation with more footage, similar to that of the well-reconstructed corners in the original reconstruction. 

### Further Investigation

As I touched on before, I do plan on investigating a better approach to targeting a particular location in the room with camera intrinsics and using SLAM to know where exactly the drone is in the room relative to everything else. But beyond that, the investigations in this project left me wondering about a few more questions that I want to explore. Namely, what is the best way to detect places in need of refinement in a point cloud? I was just using local sparsity, but I can imagine that this would fail to pick up incorrect representations -- it only finds non-existent ones. Also, what would the most ideal flight path be for surveying the room? What if I used object detection to plan such a path? But even then, how could I plan an ideal path given some objects I want to capture? Finally, I want to objectively rate my effectiveness of reconstruction against other techniques like LiDAR and photogrammetry to see if this could actually hold value in real-world reconstructions. 

### Conclusion

This was a great learning opportunity for me, intersecting my interest in drone vision with real-world use cases. I'm proud of myself for demonstrating the potential of leveraging accessible and affordable tools to tackle complex problems in 3D reconstruction, with far-reaching applications across defense, security, and numerous other industries. By utilizing a low-cost DJI Tello drone and an innovative workflow integrating Gaussian reconstruction and point cloud manipulation, this work lays the foundation for a future where precise, real-time scene mapping is achievable for anyone. 

Beyond the technical contributions, this project aligns with my broader vision of using engineering to solve pressing real-world challenges—whether it's ensuring safety in military operations, protecting endangered wildlife, or enhancing urban planning and architectural preservation. The possibilities unlocked by democratizing advanced 3D reconstruction technologies are vast, and this project represents a step forward in making those possibilities a reality.


[ghlink]: https://github.com/siddshashi/BedroomReconstruction 