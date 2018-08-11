---
title: "Camera"
date: 2018-07-05T15:43:48+08:00
lastmod: 2018-07-06T10:50:48+08:00
draft: false
tags: ["camera", "filters", "glsl"]
categories: ["hardware"]
author: "Maksim Surguy"
description: "Learn how to use camera with Processing on the Pi"
thumbnail: "thumbnail.jpg"
---

# Introduction

Since its first release, Processing has been known for its capacity in creating visualizations. It's strength in manipulating pixels of images enables more experimentation when external image sources, like cameras, are used.

While interesting and meaningful, using the built-in camera of the laptop or desktop computer with Processing can be limited by the form factor and the input methods of the computer. The portability and expandability of Raspberry Pi single board computers opens up new frontiers for using camera as input for Processing sketches.

The combination of Processing, camera, and a couple of components connected to Pi's GPIO could be used to make some unique experiences while remaining affordable. Think of possibilities like:

- Portable cameras with filters that are controlled by physical buttons and knobs
- Portrait booths that generate artwork based on recent snapshot
- Computer Vision experiments
- Timelapse rigs
- and more

(Video of some filters / sketches in action?)

Of course this is just a short glimpse of what's possible. The knowledge you gain in this tutorial should enable you to create your own projects using camera input in Processing on Raspberry Pi.

Let's take a look at what you will need to have in order to make the projects in this tutorial.
  
## Required Materials

The main component that you would need for this tutorial is the camera attached to Raspberry Pi. Below is the full list of parts necessary for this tutorial:

- a Raspberry Pi model 3+, 3 or 2 (those are recommended, it will work the Pi Zero and older versions, albeit much more slowly) with Processing [installed](https://pi.processing.org/get-started/)
- TV or any screen / monitor with HDMI input
- Official Raspberry Pi Camera or a USB Webcam compatible with Raspberry Pi

Optional:

- 1 push button
- Wires

{{% message type="warning" title="A note about cameras" %}}
The official Raspberry Pi Camera is recommended because some inexpensive alternatives have been known to not work well with Processing. Also, if a USB webcam is used instead of the Pi Camera, there might be slight performance issues.  
{{% /message %}}

## Overview of using camera with Processing on the Pi

Getting the video frames from camera in Processing has to be facilitated by an external library. The official [Video Library](https://processing.org/reference/libraries/video/) works well on Windows, Mac and some linux distributions. Unfortunately, the official Video Library does not work on Raspberry Pi running Raspbian operating system. 

Thanks to the hard work of Gottfried Haider(TODO: add more details?), there is a replacement for the Video Library that works on the Pi, and that is [GL Video Library](https://github.com/gohai/processing-glvideo)

# GL Video library

[GL Video Library](https://github.com/gohai/processing-glvideo) works well on Raspberry Pi computers running Raspbian OS. The library can be installed through the Library Manager and it enables you to:

- Capture frames from camera via GLCapture subclass
- Read frames from video files via GLMovie subclass

TODO: which other details are necessary? Hardware acceleration? Etc

## Installation and set up

To use GL Video Library in Processing on the Pi, find it in the contribution manager and install it:

{{< figure src="library-manager.png" title="Installing GL Video library" >}} 

Now, let's connect a camera to your Pi and set it up. There are two types of cameras that GL Video can work with:

- Raspberry Pi Camera (recommended)
- USB webcams

The setup will be different depending on the type of camera so let's go over these two options:

### If using a webcam

If a USB webcam is used, no other setup is necessary. Just plug the camera in and you're good to go! Keep in mind, USB webcams might deliver lower performance than the Pi Camera.

### If using the Pi Camera

If it is the first time you Pi Camera on the Pi, some preliminary steps are needed. 

Connect the camera to the Pi. Be sure to turn off the Pi and disconnect it from power before connecting the camera. After the camera is connected, boot up the Pi and enable the camera interface in `raspi-config` tool:

{{< figure src="raspi-config.png" class="center"  title="Enabling the camera interface on the Pi" >}} 

When a Raspberry Pi Camera is used, GL Video library needs a special driver to be enabled on the operating system level. Add the line `"bcm2835_v4l2"` (without quotation marks) to the file `/etc/modules`. After a restart you should be able to use the camera in Processing.

With the library downloaded and the camera connected / set up, we can start using GL Video class to work with the video stream from the camera. The `GLCapture` class within GL Video is the class that we'll be working with.

## Using GLCapture class

The main purpose of the `GLCapture` class is to set up framerate and resolution of the camera, and to read the data from the camera as an array of pixel values. `GLCapture` class works with P2D and P3D renderers and provides methods that are very similar to the `Capture` class within original [Video Library](https://processing.org/reference/libraries/video/). 

If you've never worked with the Video Library, you are encouraged to take a look at an excellent tutorial by Daniel Shiffman that goes over the steps necessary to read a video stream from the camera in Processing: 
https://processing.org/tutorials/video/ 

The main methods that GLCapture provides, are:

- `list()` - lists all cameras connected to the computer
- `start()` - starts the video stream from camera
- `stop()` - stops the video stream from camera
- `available()` - checks if the video is available
- `read()` - populates an object with the pixel data of the video frame

{{% message type="focus" title="Difference between GLCapture and the original Capture class" %}}
Though the syntax and the purpose of the two classes are very similar, there are some subtle differences between the two. For example, the `captureEvent` callback function that is in Capture class is not in GLCapture class. In GL Video, ensuring that video data is being provided by the camera is done by using the `available()` method.

TODO: Is there difference between how P2D , P3D, and other renderers behave in these two classes? 
{{% /message %}}

Let's dig into using the GLCapture class to start capturing the video stream! The process of using GLCapture class looks like this:

- Make sure the sketch renderer is setup to be **P2D** or **P3D**
- Import the GL Video library that contains GLCapture class (`import gohai.glvideo.*`)
- Create a new GLCapture object that will stream and store the pixels from the video 
- Initialize the GLCapture object, specifying camera framerate, width and height of the desired video stream
- Start the stream via `.start()` method
- Read the video stream when it is available 

Enough with the theory. Let's try this class out in practice! The following [example sketch](https://github.com/gohai/processing-glvideo/blob/master/examples/SimpleCapture/SimpleCapture.pde) comes with the GL Video library and will serve as a building block for our next steps. Running this example will result in a window which reflects whatever the camera is capturing:

TODO: add a video demo of this example 

```processing
import gohai.glvideo.*;
GLCapture video;

void setup() {
  size(320, 240, P2D); // Important to note the renderer
  
  // Get the list of cameras connected to the Pi
  String[] devices = GLCapture.list(); 
  println("Devices:");
  printArray(devices);
  
  // Get the resolutions and framerates supported by the first camera
  if (0 < devices.length) {
    String[] configs = GLCapture.configs(devices[0]); 
    println("Configs:");
    printArray(configs);
  }

  // this will use the first recognized camera by default
  video = new GLCapture(this);

  // you could be more specific also, e.g.
  //video = new GLCapture(this, devices[0]);
  //video = new GLCapture(this, devices[0], 640, 480, 25);
  //video = new GLCapture(this, devices[0], configs[0]);

  video.start();
}

void draw() {
  background(0);
  // If the camera is sending new data, capture that data
  if (video.available()) {
    video.read();
  }
  // Copy pixels into a PImage object and show on the screen
  image(video, 0, 0, width, height);
}
```

There are a couple important things from this code that will save you a lot of headache later: 

- Listing connected cameras and camera capabilities 
- Using framerates and resolutions supported by the cameras you're using

### Listing the cameras connected to the Pi 
Sometimes you might want to have more than single camera connected to the Pi. You could list all cameras and use specific camera connected to the Pi by using `GLCapture.list()` method:

```processing
String[] devices = GLCapture.list(); 
println("Devices:");
printArray(devices);
...
firstVideo = new GLCapture(this, devices[0]);
secondVideo = new GLCapture(this, devices[1]);
```

To get an idea of the framerates and resolutions supported by the camera(s), you can use `GLCapture.configs()` method.

### Finding out camera capabilities

For each camera connected to the Pi, it might be useful to know what capability they provide. Using `GLCapture.configs()` method should return all possible combinations of resolutions and framerates that you camera supports:

```processing
if (0 < devices.length) {
    String[] configs = GLCapture.configs(devices[0]); 
    println("Configs:");
    printArray(configs);
  }
```

## Simple capture

Simple capture

https://github.com/processing/processing-video/tree/master/examples/Capture  
  
# Mini projects with the camera

## color picker from GLVideo

## Histogram preview

## Selfie with Processing image filters (blur, threshold, etc)

### adding a button for shutter



## Using GLSL shaders

Because the data we get from GL Video library is essentially regular pixel data, we can do whatever we want with those pixels after putting them onto a PImage. For example, we can take advantage of using hardware accelerated shaders to offload image processing from the relatively slow CPU and onto the graphics processing unit (GPU) of the Raspberry Pi. 



# Next steps