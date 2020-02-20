# Ray Tracing in NumPy
The algorithm for ray tracing is simple, but computationally expensive compared to many other rendering methods. Since, combined with the relative slowness of python compared to C# which rendering engines are normally written in, the goal of this project is not a fast renderer. 

## Overview
The program calculates n samples per pixel. For each sample and pixel, a ray is sent out from the camera through that pixel. It its checked for intersection with any objects in the scene. If none is found, the pixel is given 0 value. 

If intersection is found, bounces off objects are calculated, up to 4, with each absorbing an amount of light. If, before all bounces are used up, the ray hits a light, the pixel is given a value equal to (1-bk). b is the number of bounces and k is a constant representing how much light is absorbed at each bounce. Per material absorption coefficients are not yet implemented.

Surface roughness is approximated by adding a "deflection" vector to the exact reflection vector, generated from a random 2D gaussian rotated to be tangent to the surface. This gaussian is scaled by a parameter called "roughness". The smaller the value, the more specular the reflection. This is a crude approximation to the roughness methods used in PBR (physically based rendering) shaders.

![image.png](https://i.postimg.cc/tCvh04jt/image.png)

512x512 pixels, 40 samples, 4 bounces. time taken: 1.5s per sample. 

## Features

### Multi-bounce 
A light ray can bounce as many times as are set by the program. While n bounce passes is O(n) complexity at the moment, light rays that have not intersected anything on the first pass will never intersect anything. So in theory they can be removed from the calculations. However since all ray paths are calculated together as arrays this is not trivial. 


### Z depth
The same property that makes it possible to increase the n bounce calculations also allows z-depth to be trivial. If a ray directly from the camera never intersects anything, it never will, and neither will any other rays from that pixel. In fact, the distances from the camera to the first intersection are the same for each pass, as the randomness only comes during reflection.

Z depth can be calculated from the distances already found while finding intersections, making a very cheap pass to calculate. Below is a plot of log(Zdepth) for the default scene. The logarithm was taken as the scene extends to infinity, and so the difference in distances between the front of the sphere and back of the sphere are very small. This is a problem in other engines like Blender's Cycles, where it is solved by allowing the exclusion of the background from the calculations.

![image.png](https://i.postimg.cc/YSms2c44/image.png)
 
### Global Illumination
To provide a more "natural" scene, light can be allowed to come from areas where there are no light objects. This mirrors the fact that on earth, due to the atmosphere, it is very rare for areas in shadow to be receiving no light at all. There is always light coming for every direction, even if it is quite weak. This can be implemented after all bounce passes have been calculated, buy letting rays that never met a light to still illuminate, with the amount of illumination decreasing with each  bounce required for them to reach a path that never intersects anything. 

[![image.png](https://i.postimg.cc/d0wcftNs/image.png)](https://postimg.cc/68jP2wNg)

## TODO
###Ambient Occlusion
Ambient occlusion (AO) is a rendering pass often used to reduce complexity and increase "realism" in real time rendering. It is cheap and so is often used to approximate the accuracy of ray tracing when using less sophisticated rendering techniques.  It simulates the darkness of concave corners under global illumination. This can be calculated by looking at the change in Z depth near each pixel, to see how the scene geometry changes, and how occluded it should be (Screen Space AO). 
###General Mesh
Most objects that are rendered are not stored as parametric equations like the sphere or plane in the default scene. Instead, they are stored as arrays of vertices, lines and faces that make up solid objects. The most fundamental object is then the triangle, 3 vertices jointed by 3 lines to make a single face. During modelling, surfaces are built out of these triangles, or more often squares that are then further subdivided into triangles when the model is finished. Each triangle has a surface normal that defines the direction of reflection for a light ray, as with the parametric objects. However, for smooth objects like a sphere or a human, an unacceptable number of triangles are needed to achieve a surface that appears smooth. Instead, smooth shading stores a normal at every vertex instead, oriented in the direction the normal of the true smooth surface that is being approximated. This allows the normal at every point on a face to be calculated via linear interpolation of the vertex normals.
