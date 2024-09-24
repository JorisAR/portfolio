---
title: "Real-Time Zelda inspired Isoline Map"
featured_image: '/images/IsoContourMap/teaser.png'
description: "Depth Rendering and Post Processing"
date: "2024-09-01"
---

{{< figure src="/portfolio/images/IsoContourMap/teaser.png" width=100% >}}

In 3D virtual environments, one can benefit greatly from having a map to lead them the way. Oftentimes maps are created by talented artists. However, when procedurally generating a terrain we have to create a map programmatically. 

In our case, the goal is to generated a map for our procedurally generated [Smooth Voxel Terrain]({{< ref "/posts/SmoothVoxelTerrain.md" >}} "Smooth Voxel Terrain"). I planned out two possible solutions:

1. Generate a heightmap texture based on the octree. This would involve raymarching through the SDF for every pixel in the texture.
2. Use a second camera to capture a depth texture of the world from above.

I chose to implement the second solution, as its relatively simple, cheap to compute and highly customizable. In fact, it is fast enough to be used in realtime, as it fully uses the efficient rasterization pipeline modern GPUs are optimized for. The only major drawback is that we are required to have generated the terrain before being able to render the minimap, thus introducing a dependency on terrain generation.

## Implementation

It's a fairly straightforward process to generate the map, but it takes a couple distinct steps regardless. I have implemented it in Godot using C#, but the ideas should be generally applicable.

### Setup
We set up a camera way up in the sky, pointing downward to the sky. In my case, I decided to move it to **y = 2048**, with the horizontal position being equal to that of the player. I chose to use a perspective camera rather than an orthographic one, which means we have to calculate a suitable FoV. To do this, we can decide a height to focus the on, I chose zero. We can use the following code to adapt the camera's frustum to be the given width at our chosen height. 

```cs
        private void UpdateCameraFoV(float width)
        {
            const float focusHeight = 0;
            var cameraHeight = _camera3D.GlobalPosition.Y;
            var angle = 2 
            * Mathf.Atan(width / (2 * (cameraHeight - focusHeight))) 
            * 180 / Mathf.Pi;
            _camera3D.Fov = Mathf.Clamp(angle, 1.0f, 179.0f);
        }
```

When rendering to a texture, we get the following result: (at a width of 250)

{{< figure src="/portfolio/images/IsoContourMap/step1.png" width=100% >}}

*Specifically for Godot, I use a [SubViewPort](https://docs.godotengine.org/en/stable/classes/class_subviewport.html) to render a secondary camera to a texture, which I display in the UI using a [TextureRect](https://docs.godotengine.org/en/stable/classes/class_texturerect.html)*

### Depth

However, we need the height of the terrain at each pixel of the map rather than the color. Godot unfortunately does not allow you to easily render auxiliary depth textures from arbitrary locations. Instead we can use a costlier option: 
- Render the opaque scene to a second camera.
- Place a large quad in front of the secondary camera, with a shader that transforms camera depth into fragment height.

From the primary camera it looks rather interesting:
{{< figure src="/portfolio/images/IsoContourMap/skyquad.png" width=100% >}}

*Normally the quad would be hidden from the primary camera either using render layers, or by the fact that we underside of the quad is culled.*


We render the quad using the following fragment shader, mostly taken from the [Godot Docs](https://docs.godotengine.org/en/stable/tutorials/shaders/advanced_postprocessing.html). We can scale the height to be in a more easily observable range by remapping the height between -50 and 50 to 0 and 1.

```glsl
void fragment() {
	float depth = texture(DEPTH_TEXTURE, SCREEN_UV).r;
	vec3 ndc = vec3(SCREEN_UV * 2.0 - 1.0, depth);
	vec4 world = CAMERA * INV_PROJECTION_MATRIX * vec4(ndc, 1.0);
    vec3 world_position = world.xyz / world.w;
	float height = world_position.y;
	
	const float maxHeight = 50.0f;
	const float minHeight = -50.0f;
	ALBEDO = vec3((height - minHeight) / (maxHeight - minHeight));
}

```

Our map looks like this now:

{{< figure src="/portfolio/images/IsoContourMap/step2.png" width=100% >}}


### Discretizing height.

In Computer Graphics, we use Illustrative Rendering techniques to aid people in understanding certain data. Humans are not great at interpreting a linear color gradient, as we perceive luminance non-linearly. This is a problem for our heightmap, as we want players to be able to make informed decisions about the height and gradient of a terrain at a certain position.

[Isoline maps](https://en.wikipedia.org/wiki/Contour_line) have been used for centuries to display the height particular sections of land, which makes them very suitable for our use case. We can achieve a convincing isoline look very easily using a two step process. First, we expand our fragment shader to discretize the height into distinct color bands. Like in zelda, I chose to split the map into two regions, water and land, based on height. I wrote the following GLSL function to achieve it, were isoDist is the bandwidth of height values per color band.


```glsl
const float isoDist = 5.0;
const float isoDistInv = 1.0 / isoDist;

vec3 terrainColor(float height) {
	const float waterMin = -100.0;
	const float waterCutoff = -55.0;
	const float terrainMax = 512.0;

	float t = clamp(height > waterCutoff ?
		round((height - waterCutoff) * isoDistInv) / (terrainMax - waterCutoff) * isoDist:
			round((height - waterMin) * isoDistInv) / (waterCutoff - waterMin) * isoDist, 0.0, 1.0);

	return height > waterCutoff ? mix(terrain_color.rgb, terrain_top_color.rgb, t) : mix(water_color.rgb, water_top_color.rgb, t);
}
```

This results in the following map:
{{< figure src="/portfolio/images/IsoContourMap/step3.png" width=100% >}}


### Isolines.
The map is much more readable than the previous depth image, but it's still relatively hard as we do not have very clear iso-lines. To implement these, I added a custom post processing stage to the map camera. Godot 4.3 introduced a [compositor to enable custom post processing passes.](https://github.com/godotengine/godot-demo-projects/tree/master/compute/post_shader)

The most common edge detection algorithm in computer graphics is the [canny edge detector.](https://en.wikipedia.org/wiki/Canny_edge_detector) The idea is that gradients in the image are largest around edges. We calculate the gradients, and color a pixel black where the gradient is under a threshold, and white where its above. 

However in our case, we can use a much simpler technique that produces more desirable results. We can simply color a pixel slightly darker if the pixel above or next to it are a different color. This results in very clean 1 pixel wide subtle edges, but it only works because we discretized the image earlier.


```glsl
void main() {
	ivec2 uv = ivec2(gl_GlobalInvocationID.xy);
	ivec2 size = ivec2(params.raster_size);

	// Prevent reading/writing out of bounds.
	if (uv.x >= size.x || uv.y >= size.y) {
		return;
	}

	// Read from our color buffer.
	vec4 color = imageLoad(color_image, uv);
	vec4 s = imageLoad(color_image, uv + ivec2(0, 1));
	vec4 e = imageLoad(color_image, uv + ivec2(1, 0));

	color = color != s || color != e ? color * 0.8f : color;

	// Write back to our color buffer.
	imageStore(color_image, uv, color);
}

```

The resulting map looks nice, and is well interpretable:

{{< figure src="/portfolio/images/IsoContourMap/step4.png" width=100% >}}



## Conclusion

{{< figure src="/portfolio/images/IsoContourMap/result.png" width=100% >}}

The pipeline I presented today is relatively simple to implement, and should work in any environment that has the desired part of the terrain ready in a 3D scene. Best of all, it is computable in real time, so it could be easily integrated into a game as a minimap for instance.

