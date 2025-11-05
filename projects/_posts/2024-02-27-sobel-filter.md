---
layout: post
title: Sobel Filter
author: lone-faerie
date: 2024-02-27
tags:
  - c++
  - embedded
  - raspberry-pi
  - linux
  - school
---
### Sobel filter for CPE 442 - Real Time Embedded Systems
<!--more-->
[![Download](https://img.shields.io/badge/.tar.gz-download-blue?color=%23b5e853&logo=data:image/svg%2bxml;base64,{{ site.data.icons.download.base64 }})](/assets/sobel.tar.gz)
* * *
This program is designed to run on a Raspberry Pi 4B. It applies a sobel filter to a video and displays the output as it is rendered, achieving 24.2 frames per second on a 1080p input.

<div style="text-align:center;"><video width="560" height="315" controls="controls" preload="metadata"><source src="/assets/sobel.mp4" type="video/mp4"></source></video></div>
* * *
The sobel filter is written in C++ and built around the OpenCV library. Each frame is converted to grayscale using the BT709 standard and then convolved with two matrices to produce the final output.

$$G_x = \begin{bmatrix}-1 & 0 & +1\\-2 & 0 & +2\\-1 & 0 & +1\end{bmatrix}, G_y = \begin{bmatrix}-1 & -2 & -1\\0 & 0 & 0\\+1 & +2 & +1\end{bmatrix}$$

Multithreading is implemented to improve performance by splitting each frame into 4 quadrants. This is achieved using pthreads, with a worker thread for each quadrant and one main thread to coordinate the work. Two barriers are used to avoid race conditions, one for when each worker thread is done with the current frame and one for when the main thread has loaded the next frame.

To further improve performance, the program makes use of SIMD instructions, particularly ARM NEON intrinsics. Since pixels are each 8 bits, frames are processed in vectors of 8 16-bit integers before being reinterpreted back to 8 bits, as 16 bits are required to avoid overflow. Convolution is achieved using a series of multiply-accumulate operations, and the resulting to vectors for x and y are added together.

To optimize the program even further, gperftools is used to profile the code. The biggest gain in performance came from rewriting the grayscale conversion to used fixed-point arithmetic. This eliminated the overhead of converting each pixel to a floating-point value and back to an integer, as well as made use of the more efficient integer operations over the floating-point equivalents. Fixed-point arithmetic gained around 3 frames per second of performance, around a 14% improvement. Additional performance was gained through inlining functions to remove function call overhead.

{% include figure.html caption="Initial Profile" src="/assets/sobel-initial.png" %}

{% include figure.html caption="Final Profile" src="/assets/sobel-final.png" %}