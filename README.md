[//]: # (Image References)
[prius]: ./images/Prius_animated.gif "Toyota Prius"
[Wires]: ./images/wires_in_slots1.jpg "Wires"
[Circles]: ./images/circlepacking.gif "Circles"
[lanelines]: ./images/lanelines.gif "LaneLines"
[spirograph]: ./images/spirograph.png "Spirograph"
[harmonograph]: ./images/harmonograph.png "Harmonograph"
[stochasticmodelling]: ./images/stochasticmodelling.gif "Stochastic Modelling"
[swlayers]: ./images/swlayers.png "Layered approach"
[swlayers1]: ./images/RaNa_Target_Abstraction_Bootloader.png "layered approach"

# rlib #
This is an attempt to use opensource software to design electric machines. Used Scilab and FEMM for generating variety of machine designs in 2D and analyzed. Further, these models are dynamically generated to feed the the optimization routine. The optimization routine is based on random walk methods. Please visit the wiki page for rlib [HERE](https://bitbucket.org/saras152/rlib/wiki).

![alt text][Wires]![alt text][prius]

# Polygon packing #
The 2D analysis for rlib required placing the wires in the motor slots, in some cases. I attempted to create circles in polygon. The wiki for it is [HERE](https://bitbucket.org/saras152/polygon_packing/wiki/Home).

![alt text][Circles]


# Computer Vision #
The jupyter notebook code with python script for detecting lane lines on the road is [HERE](https://github.com/saras152/Finding_Lane_Lines_on_the_Road). The readme.md describes the techniques used and some of the barriers crossed.

![alt text][lanelines]


# Spirographs and Harmonographs #
Created MATLAB script to generate some images for fun! The images are [HERE](https://bitbucket.org/saras152/harmonograph/wiki).

![alt text][spirograph]
![alt text][harmonograph]

# Stochastic Modelling #
Attempted stochastic modelling for sales prediction. The wiki is [HERE](https://bitbucket.org/saras152/marketmodellingstochastic/wiki/Home).

![Stochastic Modelling GIF][stochasticmodelling]

# Embedded Systems #

![Layered approach][swlayers]
![Layered approach][swlayers1]

#### Embedded File System Drivers ####
Device Driver implementation for FAT32 File System. The corresponding wiki is [HERE](https://bitbucket.org/saras152/filesystem_fat/wiki/Home).

#### Embedded USB Host ####

Implemented the USB HOST drivers on microcontroller ( already has USB physical layer). Corresponding wiki is [HERE](https://bitbucket.org/saras152/usbhost_embedded/wiki/Home).
