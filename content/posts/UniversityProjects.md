---
title: 'University Projects'
featured_image: '/images/UniversityProjects/collage.png'
description: "A diverse selection of Computer Graphics projects."
date: "2024-08-01"
---

I started studying at TU Delft in september 2020, and I have worked on many projects for courses since then. This is a collection of my proudest work related to computer graphics from my time at university. All other projects listed on my portfolio were made in my free time.


## Pizzicato
A serious game developed in 10 weeks with 4 other peers in collaboration with Psychology researchers, during which I was Game Designer and Programmer. We used Google's Mediapipe to capture the player's hands from a regular webcam in real time. We use this input in a rhythm game we call Pizzicato. The goal of the project was to provide a basis for research into sonification, which refers to the notion that actions (from users) produce sounds (positive reinforcement). Psychology researchers hypothesize sonification can be exploited to accelerate the learning process, useful during e.g. Rehabilitation.

The game itself is a rhythm game, where the player has to pinch their fingers to pop bubbles at the right time, indicated both visually and by the background track. To aid research, its a very customizable yet enjoyable experience. We published a paper about it, and it is actively being used at Leiden for research into sonification.

{{< figure src="/portfolio/images/UniversityProjects/pizzicato.png" width=100% title="A Screenshot from Pizzicato.">}}

## Holonomy
For my bachelor thesis I worked on populating an existing infinite hyperbolic world using Wave Function Collapse, to see if it would improve immersion and have potential for improved navigational capabilities, compared to a mostly empty world. My research was done with "Holonomy" as a basis, which is an environment made for Virtual Reality headsets. It is made up of three by three tiles in the real world, but an infinite hyperbolic plane (with 5 order square tiling) in the virtual world. Solely by walking in the real world, you are able to explore an infinite virtual world, thanks to the mathematical principle of Holonomy. Due to the hyperbolic nature of the world, an object on the right side of a tree thunk cannot be observed on the left side.

During my masters I had the opportunity to continue work on this project. I mostly worked on rendering and gameplay scripting during this time. We also published a paper about it.

{{< figure src="/portfolio/images/UniversityProjects/holonomy1.png" width=100% title="A Screenshot from Holonomy.">}}


## Volume Visualization in C++
With two peers, I worked on a basic volume visualization tool in C++. During a later stage of the project, I implemented additional visualization tools to support additional render modes such as Ambient Occlusion using N-Buffers, Metallic Surfaces using a precomputed Curvature Volume, and outlines.

{{< figure src="/portfolio/images/UniversityProjects/volvis1.png" width=100% title="Ambient Occlusion">}}
{{< figure src="/portfolio/images/UniversityProjects/volvis2.png" width=100% title="Composition using a 2D transfer function">}}
{{< figure src="/portfolio/images/UniversityProjects/volvis3.png" width=100% title="Metallic Surface">}}

## 3D game engine in OpenGL and C++
With one peer, I created a simple 3D game engine in C++ and OpenGL in about a week. We added support for many lights at once (both Directional and Spotlights), HDR pipeline, post processing(Bloom, ToneMapping), simple physics including character movement, pushable cubes and a heightmap based collider, animated grass using instancing, Real-time Environment mapping, GPU particles (Fireflies), Frustum Culling, and depth based Fog.

{{< figure src="/portfolio/images/UniversityProjects/3dgame1.png" width=100% title="An interactive environment made with our engine">}}

## Image Processing
During multiple courses I worked on image processing projects, here are some examples.
{{< figure src="/portfolio/images/UniversityProjects/imageProcessing1.png" width=500 title="(Python) A computer vision pipeline to capture car license plates from an input video.">}}

{{< figure src="/portfolio/images/UniversityProjects/imageProcessing2.png" width=500 title="(C++ & OpenGL) A tool to convert images into drawings in real time using shaders, based on a paper \"Interactive painterly stylization of images, videos and 3D animations\" by Lu et. al. ">}}

{{< figure src="/portfolio/images/UniversityProjects/imageProcessing3.png" width=500 title="(Python) A tool to convert images and human drawn  scribbles resembling depth into a depth map using poisson image editing, which can be used to apply depth of field, or to generate small videos with the Ken Burns Effect.">}}

{{< figure src="/portfolio/images/UniversityProjects/imageProcessing4.png" width=500 title="(Python & Pytorch) A style transfer tool to draw an input image using a style image, such as drawing this duck in the style of Van Gogh's Starry Night.">}}

## Raytracer in C++
One of my first times using C++, I coded a raytracer with two peers. We added features such as depth of field, texture mapping, normal interpolation and area lights.  

{{< figure src="/portfolio/images/UniversityProjects/raytracer1.png" width=100% title="Cornell Box rendered using my Raytracer.">}}