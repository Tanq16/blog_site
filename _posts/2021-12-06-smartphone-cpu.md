---
title: How do Smartphone CPUs work
date: 2021-12-06 12:00:00 +0500
categories: [Computer Science, General]
tags: [fundamental,basic,hardware,soc,smartphone,cpu]
---

An SoC has 5-10 billion transistors that fit in the size of a penny. A phone microchip is composed of DRAM (Dynamic RAM) on top and the SoC on the bottom. This setup is called a Package on Package (PoP). This is technically the microchip. The separate layers of the DRAM and the SoC are each called a die. The word "chip" is meant to refer to a lot of things. Other small components or microchips along with DRAM and the SoC are all called chips. The companies that design SoCs are generally very secretive about the designs of the chip.

## All sections of the SoC

Like the human brain, the SoC is divided into many sections that handle separate data and functions. The main components are &rarr;

1. CPU containing multiple cores
2. GPU for rendering graphics
3. A shared memory cache (usually 4-8 MB)
4. DSP or Digital signal processor that interfaces with speakers, microphone, etc.
5. Display engine tat communicates with the touch screen of the display
6. The video processor that compresses and decompresses images and video and enables 4k recording and playback
7. The ISP or image signal processor that processes pictures and videos taken by cameras
8. The modem that interfaces with various wireless networks
9. The storage controller that saves and loads information from the flash storage microchip
10. The memory controller that connects to the DRAM on top of the SoC
11. The secure enclave that executes encryption and manages the public and private keys
12. Peripherals such as clocks, temperature functions, debug ports and general purpose inputs and outputs
13. An always on micro-controller unit
14. The NoC or Network on Chip that arbitrates or manages data flow through the SoC, DRAM and other parts on the board
15. A power management circuit that complements the power management performed by separate chips outside the SoC
16. A Neural Processing Unit or NPU to execute ML algorithms far more efficiently than the CPU in terms of speed and power consumption

Different generations of SoC use different names to mix in many different components for marketing. This is mainly to impress the customers but mainly all SoC have the same basic principles.

## Processing of an image on the SoC

The photons from the scene enter the lenses, flow through a color filter array and hit the sensor's photo diode filters. These color filtered photons are absorbed by each diode, which is converted into an analog current, then converted to a 12 bit binary value from each diode. A 12 MP camera has 12 million 12 bit binary values. Each pixel is either of RGB, thus must be converted to a recognizable image. Before this processing it is stored in the DRAM. For this, the data travels into the SoC through the MIPI (mobile industry processor interface), which can send or receive data at around 528Gbps. The data is then routed by the NoC arbitrator through the SoC into the memory controller and then into the DRAM. The data path between the DRAM and camera is shared by everything and it is the job of the NoC to prioritize the incoming data of the uncompressed RAW image to not lose any data from the camera. A 12 MP RAW image takes around 24 MB, but the memory cache on the SoC is 4-8 MB and is shared amongst all processes.

So, when the ISP or the Video processor work on a recently taken picture to make it viewable or to compress it, they can only work on small subsections of the image leaving the entire image temporarily stored on the DRAM. The same happens while watching a video or playing a video game or using any other app.

The next step is for the ISP on the SoC to read the RAW uncompressed data and perform a number of corrective steps. These include correcting for darker pixels on the edge of the sensor due to lens shading, performing the de-mosaicing process which involves taking the image data and the pattern on the color filter and calculate an RGB value for every pixel. Next, the ISP de-noise, sharpens, enhances and color corrects to match the RGB hue of the lens to that in the pixels. Finally, it tone maps the image. This gives a 112 million pixel image with each pixel having an 8 bit tuple for RGB values. To get the picture to promptly appear on the screen, the RGB data from the ISP is sent to the GPU where it gets overlayed into the graphics for the camera app and scaled to it the screen. The resulting RGB values get sent to the display processor and the image is routed into the display, where it is converted into current intensities to light up a pattern of RGB pixels. To save the picture on the phone, the picture must first be compressed. So the data in the DRAM is sent to a dedicated video coding processor, where it gets converted from RGB to YUV or luminance, blue chrominance and red chrominance. This undergoes a series of algorithms to remove information that is undetectable to the human eye (JPEG compression). This compresses the image into approximately 3 MB. This is sent back to the DRAM and routed to the phone's flash storage for long term storage. To send this image to a friend, the data would be routed to the modem, where it is divided into packets and sent to 4g, 5g or WiFi chips, converted into electromagnetic waves and sent to the cellular or WiFi network.

All data routed in the chip is transmitted over circuit wires which is a part of the NoC. This network has routers and switches for shared access to rotes and targets like DRAM. The circuits have different widths of wires based on what they are talking to and the amount of information that needs to be sent. Typically there are 128-256 wires running in parallel carrying 1 bit at a time and operating between 500 and 1500MHz. The frequencies of the transfers ramp up and down to conserve battery according to the requirement of applications. This is called Dynamic Frequency Scaling.

## CPU section

The CPU has multiple cores and each can run part of a program by executing instructions. There are several caches in a core (Translation Look-aside Buffers and instruction caches). Instructions such as add, multiply, load, store, compare, jump, etc. flow via front end and the actual data flows to the execution engine, where the instructions are executed on the data. ALUs, FADD, FMUL, etc. are blocks that actually perform the operations.

All smartphones use a RISC architecture. Almost all of these architectures are licensed from ARM. Apple licenses the Instruction set Architecture from ARM and use this to design processors in house. Qualcomm licenses complete blueprints called Intellectual Property cores from ARM. These cores are integrated into the SoC, sometimes keeping the design of the core as is, but more often customizing the core and rebranding the name of the CPU as Snapdragon.

## Designing and Manufacturing the SoC

Two design principles for SoC are as follows &rarr;

1. Hardware Acceleration &rarr; This deals with the fact that instead of having a single powerful CPU with a lot of cores, the chip has a variety of chips dedicated to performing specific functions. These special function blocks or hardware accelerators perform their tasks much quicker and efficiently than if they were performed by the general purpose CPU. Example &rarr; ISP block, Encoding/Decoding block.
2. Energy Efficiency &rarr; Instead of having all high performance cores, chip designers follow a big-little design structure with 2-4 big high performance cores and 4 little cores with lower but energy efficient cores. The design of the transistor has been modified since long to create energy efficient processors. The current design s called Gate-all-around FET (Field Effect Transistor).

All SoCs are manufactured by a single company called TSMC or Taiwan SemiConductor Manufacturing Company and a smaller proportion by Samsung. They are manufactured on 300 mm wide silicon wafers in factories called Fabs (Fabrication Plants). To manufacture a chip, a wafer has to go through 120-160 process steps performed by dozens of machines.

## Resource

[How do Smartphone CPUs work?](https://www.youtube.com/watch?v=NKfW8ijmRQ4)
