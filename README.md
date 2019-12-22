# Ray Tracing in NumPy
The algorithm for ray tracing is simple, but computationally expensive compared to many other rendering methods. Combined with the relative slowness of python, its clear this project is not about fast rendering (although that would be nice). 

## Overview
The program calculates n samples per pixel. For each sample and pixel, a ray is sent out from the camera through that pixel. It its checked for intersection with any objects in the scene. If none is found, the pixel is given 0 value. 

If intersection is found, bounces off objects are calculated, up to 4, with each absorbing an amount of light. If, before all bounces are used up, the ray hits a light, the pixel is given a value equal to (1-bk). b is the number of bounces and k is a constant representing how much light is absorbed at each bounce. Per material absorption coefficients are not yet implemented.

Surface roughness is approximated by adding the values from a random 2D gaussian to the axis perpendicular to the surface normal. This gaussian is scaled by a parameter called "roughness". The smaller the value, the more specular the reflection.

![image.png](https://i.postimg.cc/tCvh04jt/image.png)

512x512 pixels, 40 samples, 4 bounces. time taken: 1.5s per sample. 

## Features

### Multi-bounce 
A light ray can bounce as many times as are set by the program. While n bounce passes is O(n) complexity at the moment, light rays that have no intercected anything on the first pass will never intercect anything. So in theory they can be removed from the calculations. However since all ray paths are calculated together as arrays this is non trival. 


### Z depth
The same property that makes it possibe to increase the n bounce calculations also allows z-depth to be trival. If a ray directily from the camera never intercects anything, it never will, and neither will any other rays from that pixel. In fact, the distances from the camera to the first intercection are the same for each pass, as the randomness only comes during reflection.

Z depth can then be calculated from the distances gotten while finding intercections. 

![image.png](https://i.postimg.cc/YSms2c44/image.png)
 
 
