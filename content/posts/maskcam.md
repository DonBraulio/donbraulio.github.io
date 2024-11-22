+++
date = '2024-11-21T13:44:18-03:00'
draft = false
title = 'Python ❤️ C++ for Embedded Computer Vision and Tracking'
type = 'post'
categories = ["blogpost"]
tags = ["python","c++","computer vision","machine learning"]
+++


{{< figure src="/images/maskcam/maskcam_hardware.avif" alt="Hardware for the smart camera" caption="Assembled smart camera that runs the computer vision pipeline" width="70%" >}}

Here's an interesting Open Source project I worked on some time ago: a real-time computer vision system on a constrained embedded device. It required combining a high-level language (Python) to tackle all the requirements (remote server comm, streaming, file sharing), with the C++ components of NVIDIA's DeepStream which leverage GPU-accelerated processing.

Let's dive into some interesting technical aspects of this full-fledged video pipeline capable of object detection, tracking, video streaming, storage management, and remote control—all running in harmony on a compact embedded device. Here’s how it works.

I was the developer for this project while working at [Tryolabs](https://tryolabs.com). It was a collaboration with [BDTI](https://bdti.com) and led to a talk at [NVIDIA's GTC](https://www.nvidia.com/en-us/on-demand/session/gtcspring21-s32588/) that we delivered together with Evan Juras, the hardware engineer.

## Challenge
NVIDIA's Jetson Nano, while affordable and accessible (USD 90 at the time), is constrained in terms of memory (4GB of RAM) and CPU resources. Developing a system capable of handling multiple tasks—including computer vision, streaming, and storage management—requires efficient memory management, GPU processing, and robust orchestration of parallel processes.

{{< figure src="/images/maskcam/maskcam_jetson.png" alt="Jetson Nano SOM" caption="Jetson Nano SOM" width="30%" >}}

Pure Python solutions fall short here:
 - The Global Interpreter Lock (GIL) limits true multithreading.
 - Multiprocessing requires inter-process communication, which can add significant overhead.
 - Handling raw video frames often involves costly memory copies between GPU and RAM.

On the flip side, writing the entire pipeline in C++ would be time-intensive and inflexible. To bridge this gap, we combined Python’s development speed with C++’s performance through NVIDIA DeepStream’s Python bindings.

## Solution


DeepStream serves as the backbone of the pipeline, handling video processing and object detection using highly optimized GPU-accelerated modules. The pipeline was defined in Python, but all the heavy lifting—frame decoding, inference, and metadata generation—happens in DeepStream’s C++-optimized core.

But also, I was working at Tryolabs at the time, and we had our standout object tracking library [Norfair](https://tryolabs.com/blog/2020/09/10/releasing-norfair-an-open-source-library-for-object-tracking) which is entirely written in Python. Naturally, we wanted to use it—not only because it’s awesome but also because we had deep expertise in its inner workings and personally knew every Kalman Filter involved.

This was an interesting challenge, because we needed real-time access to the detections but without access to the video frames, which were deep down flowing through the GPU. And even more, then we needed to draw the bounding boxes of the smoothed tracker detections back in the frame.

Long story short, we were able to solve this by probing the video pipeline after the object detection model, in order to access the detections metadata. This allowed us to process the object coordinates in Python and feed them into our Norfair tracker. Then, we had to draw the tracked objects into the frame, by using Deepstream's drawing component, which basically receives a series of commands to draw different shapes into the video frames (e.g: rectangles, circles, text). This allows manipulation of those frames but without manipulating "pixels" from python, which would require costly memory transfers between GPU and device RAM.

{{< figure src="/images/maskcam/maskcam_pipeline.avif" alt="Diagram of the video pipeline that runs the object detector and tracker" caption="Video pipeline that runs the Deepstream object detector and python tracker" width="100%" >}}

In summary, this allowed us to:
 - Run an object tracker (Norfair) entirely in Python, using only the metadata (e.g., object coordinates) from the DeepStream pipeline.
 - Avoid costly memory transfers by never copying video frames back to RAM.

### Not only Computer Vision

The complete system also needed to handle a bunch of tasks aside of object detection and tracking. We had to run 5 different Python processes in order to meet all the required features for our smart camera.

So we used Python multiprocessing to orchestrate these processes:
 1. DeepStream Pipeline + Object Tracker: Runs object detection, tracks objects with Norfair, and generates metadata.
 2. Streaming Server: Streams live video with drawn detections to remote users.
 3. Video chunks saver: Saves video segments to a RAM filesystem for temporary storage, with the option to copy segments to disk on alert or remote command.
 4. Static Web Server: Allows remote users to download saved video chunks as MP4 files.
 5. Orchestrator: Coordinates the above processes, communicates with a remote MQTT server for statistics and command handling, and ensures seamless operation.

{{< figure src="/images/maskcam/maskcam_processes.avif" alt="Python processes orchestrated" caption="Multiple python processes running in parallel managed by the orchestrator" width="100%" >}}

By delegating tasks to separate processes, the system achieved high performance while maintaining the flexibility of Python. Each process could be enabled, disabled, or restarted independently, enabling robust system behavior and faster development cycle.

## Results

In just four months, this approach enabled the creation of a real-time computer vision system capable of:
 - Detecting and tracking objects on live video.
 - Streaming video with overlays to remote users.
 - Saving and managing video chunks efficiently.
 - Providing a web interface for downloading videos.
 - Seamlessly integrating with a remote MQTT server for commands and monitoring.

This project highlights the power of combining Python and C++. Python’s high-level simplicity enabled fast development and orchestration, while C++ (via DeepStream) ensured the high performance needed for video processing. Developing this system in pure C++ would have been a monumental task, especially with just one full-time engineer and a tight four-month deadline.

Instead, this hybrid approach delivered:
 1. **Faster development:** Writing orchestration logic in Python is orders of magnitude faster than C++.
 1. **High performance:** DeepStream’s optimized C++ backend handled the most resource-intensive operations efficiently.
 1. **Resource efficiency:** By using metadata and GPU, we avoided bottlenecks like memory copies or pickling.
 1. **Flexibility:** Running a Python tracker on top of a low-level inference engine for object detection.


## Learn More

For a deeper dive into the technical details and to see the full system architecture (with beautiful diagrams!), check out the [original Tryolabs blogpost here](https://tryolabs.com/blog/2021/03/17/releasing-maskcam-an-open-source-smart-camera-based-around-jetson-nano). You can also explore the complete codebase on the [GitHub repository](https://github.com/bdtinc/maskcam).

This project is a testament to the power of combining Python’s flexibility with C++’s performance. It proves that even with constrained hardware, it’s possible to achieve high performance and scalability—without sacrificing development speed.