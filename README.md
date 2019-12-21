# Ray Tracing in NumPy
The algorithm for ray tracing is simple, but computationally expensive compared to many other rendering methods. Combined with the relative slowness of python, its clear this project is not about fast rendering (although that would be nice). 

## Overview
The program calculates n samples per pixel. For each sample and pixel, a ray is sent out from the camera through that pixel. It its checked for intersection with any objects in the scene. If none is found, the pixel is given 0 value. 

If intersection is found, bounces off objects are calculated, up to 4, with each absorbing an amount of light. If, before all bounces are used up, the ray hits a light, the pixel is given a value equal to (1-bk). b is the number of bounces and k is a constant representing how much light is absorbed at each bounce. Per material absorption coefficients are not yet implemented.

Surface roughness is approximated by adding the values from a random 2D gaussian to the axis perpendicular to the surface normal. This gaussian is scaled by a parameter called "roughness". The smaller the value, the more specular the reflection.

![image.png](https://i.postimg.cc/tCvh04jt/image.png)
512x512 pixels, 40 samples, 4 bounces. time taken: 1.5s per sample. 
 
