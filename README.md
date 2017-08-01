[//]: # (Image References)
[Wires]: ./images/wires_in_slots1.jpg "Wires"
[Circles]: ./images/circlepacking.gif "Circles"
[lanelines]: ./images/lanelines.gif "LaneLines"
[spirograph]: ./images/spirograph.png "Spirograph"
[harmonograph]: ./images/harmonograph.png "Harmonograph"
[stochasticmodelling]: ./images/stochasticmodelling.gif "Stochastic Modelling"
[swlayers]: ./images/swlayers.png "Layered approach"
[swlayers1]: ./images/RaNa_Target_Abstraction_Bootloader.png "layered approach"

I have many areas of interest - both technical and non-technical, just like everyone else. Below are some of the areas that I worked on which can be showcased. 

###### My email: hex2ascii(0x72 0x61 0x67 0x68 0x75 0x2e 0x65 0x6c 0x65 0x63 0x40 0x67 0x6d 0x61 0x69 0x6c 0x2e 0x63 0x6f 0x6d) (written in hex just to avoid detection by web crawlers) ######



## ToC ##

[Electromagnetics](#electromagnetics)

[Embedded Systems](#embedded-systems)

[Computer Vision](#computer-vision)

[Statistical Modelling](#statistical-modelling)

[Fun with Shapes](#fun-with-shapes)


# Electromagnetics #

### Electric Machine Design ###

I like open source software. I attempted to use [Scilab](https://www.scilab.org/) and [FEMM](http://www.femm.info/wiki/HomePage) to build my personal tool to generate variety of elecitrc machines of BLDC Motors with surface mounted magnets (inner rotor, outer rotor), IPM Motors (inner rotor so far) and Synchronous Reluctance Motors (inner rotor so far). Also implemented parallel FEA computations for faster characterization of the motors. These models can be dynamically generated and can be fed to the optimization routine that I implemented based on random walk methods. I call this tool "rlib".  Please visit the wiki page for rlib [HERE](https://bitbucket.org/saras152/rlib/wiki).

![alt text][Wires]
<img src="./images/Prius_animated.gif" width="170" alt="Toyota Prius Traction Motor - |B| vs theta">

### Magnetic Gears ###

This is my attempt at magnetic gears with FEA simulations. Wiki page is coming up.

# Embedded Systems #

![Layered approach][swlayers]![Layered approach][swlayers1]

### Embedded File System Drivers ###
Device Driver implementation for FAT32 File System. The corresponding wiki is [HERE](https://bitbucket.org/saras152/filesystem_fat/wiki/Home).

### Embedded USB Host ###
Implemented the USB HOST drivers on microcontroller ( already has USB physical layer). Corresponding wiki is [HERE](https://bitbucket.org/saras152/usbhost_embedded/wiki/Home).

# Computer Vision #

I tried to work with OpenCV and tried to detect the lanelines on the road. In the wiki page linked below, i tried to address few issues solved and listed possible improvements. The jupyter notebook with preliminary implementation of the algorithm using python script for detecting lane lines on the road is [HERE](https://github.com/saras152/Finding_Lane_Lines_on_the_Road). 

![alt text][lanelines]

# Statistical Modelling #
I attempted stochastic modelling for sales prediction. The wiki below also include a paper I tried to make out of the work done for this. The wiki is [HERE](https://bitbucket.org/saras152/marketmodellingstochastic/wiki/Home).

![Stochastic Modelling GIF][stochasticmodelling]

# Fun with Shapes #

## Polygon packing ##
The 2D analysis for rlib required placing the wires in the motor slots, in some cases. I attempted to create circles in polygon. The wiki for it is [HERE](https://bitbucket.org/saras152/polygon_packing/wiki/Home).

![alt text][Circles]

## Spirographs and Harmonographs ##
Created MATLAB script to generate some images for fun! The images are [HERE](https://bitbucket.org/saras152/harmonograph/wiki).

![alt text][spirograph]
![alt text][harmonograph]

