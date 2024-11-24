---
title: "Simulating Forest Fires"
featured_image: '/images/ForestFires/teaser.png'
description: "VFX using Textures, Geometric Data Processing and Billboards"
date: "2024-11-24"
---

{{< youtube N6fbC7LbrZc >}}

Inspired by [the Scintilla paper](https://storage.googleapis.com/pirk.io/projects/scintilla/index.html), I wanted to create interesting forest fire simulations on my procedurally generated [Smooth Voxel Terrain]({{< ref "/posts/SmoothVoxelTerrain.md" >}}).

To achieve it, I came up with the following requirements:
1. We need to come up with a system to modify the grass on the terrain. It needs to be able to visually indicate that it has been burnt, and in that case, it shouldn't be able to be burnt again.
2. Trees should be able to be cut in half. This way, fire could burn trees and let them fall over once they are burnt up.
3. There should be a system that renders and spreads fire in sensible ways.


These requirements combined will provide us with a system that is able to somewhat effectively simulate forest fires. Keep in mind that my goal here is not realism, but rather a proof of concept to show that my large terrain system can support such fire simulations.

## Burnable Grass
Grass in my system is rendered using gpu instancing. This means we cannot have fine control over every single grass blade, as that would introduce huge performance implications. 

Instead, we can use a texture to store essentially a lookup table of the grass state in an area. The world is already divided into chunks, so we can use one such texture per chunk.  This means that to affect grass in a particular area, we simply find the relevant chunks that contain that area, and draw the desired state to the textures stored within.

{{< figure src="/portfolio/images/ForestFires/texture.png" width=100% >}}


We can calculate texture space coordinates using the coordinates of the grass vertices relative to that chunk. In the shader, we retrieve the value from the texture, and use it to multiply the local vertex position in the y dimension, and to interpolate the fragment color between the grass color and black.



## Cuttable Trees
The idea is that the tree mesh can be split in two given an arbitrary plane. As this is done programmatically, there is some computational overhead every time a cut is made, but it saves time creating assets and enables better control for the player.

The algorithm goes as follows. First, we iterate over all vertices in the mesh, and discard those that are not in the permitted vertex color range. This enables us to discard small branches and leaves from the cut tree. We also split the vertices into two distinct groups, based on which side of the plane they are. 
Then, we iterate through all the edges that intersect the plane. For every intersection, we create two vertices, one for either side of the plane. 

Now  we need to triangulate the hole that’s left. If the contour of the original mesh on the plane is convex, such as an octagon, we can simply add an additional vertex at the centroid of the vertices that make up this contour on either side, and trivially triangulate. A different method is required for concave and disconnected contours, but I have not implemented this as nearly all contours will be convex in my case.

{{< figure src="/portfolio/images/ForestFires/cut_tree.png" width=100% >}}

Then, to make the cut logs fall over, I assign the resulting mesh to a rigidbody. I approximate the mass and inertia by calculating the volume and inertia for a tight bounding box around the log. The center of mass is at the centroid of the vertices.


Finally to complete the effect, we take the leaves that were discarded in the first step, and create a simple animation. Similar to the grass, we multiply the y coordinates of the vertices by a factor animated using a tween. Additionally, I use dithering to fade out the leaves without using expensive transparent rendering. Now it looks like the leaves are falling off the tree rather than disappearing in thin air.



## Fire System
The previous population systems are now ready for a fire system. The fire uses an incredibly simple propagation strategy. Every physics tick, the flame has a small probability to propagate. If so, I use polar coordinates to generate a random position in the xz plane around the flame. Then, I cast a ray to find the ground height at this position. If the grass is not burnt here, a new flame is placed at this position. After a couple seconds, a flame dies out. To prevent uncontrolled exponential growth, flames can only place new ones up until a certain depth, where propagated flames inherit the depth of the flame they originate from plus one.

{{< figure src="/portfolio/images/ForestFires/fire_placement.png" width=100% >}}


To render the fire, I decided to use billboards with some tricks.
Billboards are essentially quads that always face the camera, and they are often used to represent volumetric elements such as particles, or in world UI elements such as healthbars. 


The difficult part is getting the right transformation for this quad. Recall that the rasterization pipeline uses a model view projection transformation to render objects at the correct screen positions. Our goal is to align a quad’s orientation with the camera’s view direction, centered at the object’s position. 

Essentially, we want the quad to be in the xy plane after the MVP transformation has been applied. Assuming in local coordinates this is already the case, we can simply replace the first three columns of the model matrix with the ones from the inverse view matrix. This essentially un-rotates the quad, because the top left 3 by 3 part of the matrix describes rotation and scaling. The last column remains the same as it describes the translation, which we want to keep.

For the fire effect I made the decision to use alpha cut. This is less realistic, but it saves us from having transparency sorting issues. Next, we can use the dot product of the view angle and the up direction to interpolate between two shapes. This already makes the quad appear more like a 3d volumetric object.

Then, we add some noise and scroll it. We could use 2d noise, but when scrolling this upwards in texture space, it looks awkward when viewed from above. Instead, we can use a 3d noise volume. 

To sample this volume, we can use something that's similar to multi planar reformation often seen in medical visualization. The idea is that we want to essentially show a cross section of the volume on our billboard, corresponding to the plane in global space described by it. 

To get 3d points on this plane as a function of UV, we can use the following function:

$$
p(UV) = \begin{bmatrix} 0.5 \\ 0.5 \\ 0.5 \\ 0 \end{bmatrix} - \text{VIEW\_MATRIX}^{-1} \cdot \begin{bmatrix} \text{UV}_x - 0.5 \\ \text{UV}_y - 0.5 \\ 0 \\ 0 \end{bmatrix}
$$

Admittedly, the effect is a little less effective when the camera moves quickly in the axis of greatest temporal movement in the volume, but it beats scrolling 2D noise in my opinion.

Trees can also be set on fire, but it would be wasteful to instantiate many flames on it to make it look good. Instead, I simply scroll a noise texture over the tree, and use this as an alpha mask to blend between fire and the tree itself.


## Conclusion

{{< figure src="/portfolio/images/ForestFires/fire.png" width=100% >}}

Overall, these effects combined result in a fairly reasonable forest fire simulation. For more realism, photorealistic fire textures could be used, in combination with a fire spreading algorithm that takes into account the flammability of different materials. For more large-scale simulations, a more efficient fire system is required, for instance using GPU instancing combined with a central propagation system that handles all the flames in one go. 
