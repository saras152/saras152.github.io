[//]: # (Image References)
[Wires]: ./images/wires_in_slots1.jpg "Wires"
[Circles]: ./images/circlepacking.gif "Circles"
[squares]: ./images/rectangular_fitting.png "squares"
[lanelines]: ./images/lanelines.gif "LaneLines"
[lanes]: ./images/lanefinding.gif "lanes"
[spirograph]: ./images/spirograph.png "Spirograph"
[harmonograph]: ./images/harmonograph.png "Harmonograph"
[stochasticmodelling]: ./images/stochasticmodelling.gif "Stochastic Modelling"
[swlayers]: ./images/swlayers.png "Layered approach"
[swlayers1]: ./images/RaNa_Target_Abstraction_Bootloader.png "layered approach"

I have several areas of interest - both technical and non-technical, just like everyone else. Below are some of the projects related to Machine Learning field, that I have spent time on. 

e-mail: learn@redarm.in



## ToC ##
{:toc}





# Machine Learning #

### Vehicle detection and tracking

Used hog, spacial binning and color histograms features along with the Support Vector Machine module to train a linear classifier to detect the cars in an image. Used the same pipeline to implement the vehicle tracking in a video. Corresponding wiki is [HERE](https://github.com/saras152/myVehicleDetection)

![multiple vehicles](./images/trackingvehicles.gif)

### Computer Vision - Lane Finding ###

Used OpenCV's functionalities for camera calibration, distortion correction, perspective views and  warping, in combination with few  curve fitting functions to generate lane geometries from boundaries to identify the curvature and vehicle offset from lane center. The corresponding wiki is [HERE](https://github.com/saras152/myAdvancedLaneFinding).

![lane lines][lanes]

### Behavioral Cloning - driving the autonomous car in a simulator ###

Used Keras to simulate the autonomous car to make it run around the track without any accidents. The corresponding wiki is [HERE](https://github.com/saras152/myBehavioralCloningProject).

![difficult corner](./images/autonomousSim_turn.gif)

### Traffic Sign Classification ###

Used TensorFlow to classify few random images from the internet using the network trained using german traffic sign benchmarks. The corresponding wiki is [HERE](https://github.com/saras152/Traffic-Sign-Classifier).

### Computer Vision - Lane line identification ###

Used OpenCV to detect the lanelines on the road. The corresponding wiki is [HERE](https://github.com/saras152/Finding_Lane_Lines_on_the_Road). 

![alt text][lanelines]

