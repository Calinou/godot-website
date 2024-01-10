---
title: "Introducing HDDAGI, the real-time global illumination system to succeed SDFGI"
excerpt: "Godot is about to get a real-time 3D global illumination system known as HDDAGI. This article describes why it was developed as a replacement of the existing SDFGI, how it works and future steps."
categories: ["progress-report"]
author: Hugo Locurcio
image: /storage/blog/covers/introducing-hddagi-real-time-global-illumination.jpg
date: 2024-01-03 16:00:00
---

Godot is about to get a real-time 3D global illumination system known as HDDAGI. This article describes why it was developed as a replacement of the existing SDFGI, how it works and future steps.

## What is HDDAGI?

HDDAGI (Hierarchical Digital Differential Analyzer Global Illumination) is a new technique for real-time global illumination coming to **Godot 4.3**.

It's important to stress that HDDAGI is designed to be **fully compatible** with SDFGI. While the visuals differ, it relies on the same properties in the Environment class as SDFGI did. Compatibility handlers have been added: this way, using `sdfgi_*` properties in a scene or script will redirect you to the `dynamic_gi_*` properties automatically.

The high-level design goals of HDDAGI remain the same as SDFGI:

- **Real-time, no baking.** Tick a checkbox and it just works. This makes it suitable for procedurally-generated levels.
- **No high-end, hardware-accelerated raytracing required.** As long as the GPU supports Vulkan or Direct3D 12, it can be used. The algorithm is designed to run well on mid-range GPUs along with modern desktop integrated graphics.
- **Supports dynamic lights**, allowing for day/night cycles, moving flashlights, etc.

## Why replace SDFGI with HDDAGI?

Godot 4.0 introduced [SDFGI](/article/godot-40-gets-sdf-based-real-time-global-illumination) (Signed Distance Field Global Illumination), which provides real-time global illumination with no baking required.

You can find a technical document about SDFGI written by Juan Linietsky here: [Solving the accessible Global Illumination problem in Godot](/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_design_document.pdf)

The internals are also different: SDFGI works with signed distance fields, while HDDAGI uses voxels.

However, several issues became apparent with SDFGI. These issues were not fixable without a complete rewrite due to real-time signed distance field generation being expensive, especially when complex geometry is involved.

*Screenshots below use Jamsers' [Bistro-Demo-Tweaked](https://github.com/Jamsers/Bistro-Demo-Tweaked) project.*

### Improved lighting quality

HDDAGI features a new probe filtering option that is enabled by default. Probe filtering allows indirect lighting to be more evenly distributed, resulting in a smoother appearance. Thanks to probe filtering, splotches in indirect lighting are now much less common, especially when a light or the camera moves:

**SDFGI:**

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_light_update.mp4?1" type="video/mp4">
</video>

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_camera_update.mp4?1" type="video/mp4">
</video>

**HDDAGI:**

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_light_update.mp4?1" type="video/mp4">
</video>

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_camera_update.mp4?1" type="video/mp4">
</video>

Probe filtering can be disabled for a slight performance boost, or if you want to maximize the detail visible in indirect lighting.

The default number of frames used to converge indirect light has also been decreased from `30` to `12` by default, leading to faster light updates (which is beneficial when lights or the camera moves fast). It can still be adjusted in the Project Settings.

### Improved reflection quality

The voxel-based nature of HDDAGI leans itself well to real-time reflections. Compared to SDFGI which used a hacky way to obtain lighting from the signed distance field, HDDAGI properly filters voxels. This means what you see in reflections is more faithful to the actual geometry.

When comparing the debug drawing modes of SDFGI and VoxelGI, notice how SDFGI has a black outline that is not present with HDDAGI:

**SDFGI:**

![SDFGI debug draw mode](/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_debug.webp)

**HDDAGI:**

![HDDAGI debug draw mode](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_debug.webp)

For comparison, this is the actual scene being reflected (colors appear different due to environment lighting and tonemapping):

![Scene being reflected](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_reflection.webp)

Here is another example in a more detailed scene, with the filtered reflection on the right:

![HDDAGI scene vs filtered reflection](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_scene_vs_reflection.webp)

Additionally, reflection quality was improved thanks to a [more accurate normal buffer](https://github.com/godotengine/godot/pull/86316) now being used in shaders. This is particularly noticeable on objects with low roughness and smooth geometry, or when the camera rotates slowly. The improved normals make any kind of reflections appear less blocky, not just reflections provided by HDDAGI. User-provided shaders that use `normal_roughness_texture` also benefit from this change.

**Before (left), after (right):**

![Improved normals](/storage/blog/introducing-hddagi-real-time-global-illumination/improved_normals.webp)

Thanks to the new **DynamicGI > Filter > Reflection** property, you also get more control over how reflections look compared to SDFGI. By default, the filter is disabled, which allows reflections to get very crisp (albeit blocky). If this doesn't mesh well with your project's art direction, you can choose to enable the filter for smoother reflections that hide the blocky nature of voxels.

**Reflection filter disabled (left), reflection filter enabled (right):**

![HDDAGI reflection filter comparison](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_reflection_filter.webp)

### Improved performance, especially when the camera moves fast

While HDDAGI features generally improved performance in still scenes compared to SDFGI, the biggest area of improvement was in camera updates. Generating a voxelized version of the scene is *much* faster than generating a signed distance field in real-time, even for complex geometry.

Compare the FPS counter when moving the camera fast with SDFGI and HDDAGI:

**SDFGI:**

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_motion_performance.mp4?1" type="video/mp4">
</video>

**HDDAGI:**

<video controls muted>
  <source src="/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_motion_performance.mp4?1" type="video/mp4">
</video>

This improvement makes HDDAGI more viable than SDFGI for games where the camera moves fast, such as racing games, fast-paced shooters, etc. Using a lower cell size and greater number of cascades is also more viable than before. This makes HDDAGI more *scalable* in the sense that it offers options to scale from integrated graphics to high-end dedicated GPUs, including [offline rendering](https://docs.godotengine.org/en/stable/tutorials/animation/creating_movies.html).

Additionally, memory usage has been significantly reduced. This improves performance on GPUs constrained by memory bandwidth such as integrated graphics.

### Reduced light leaking

While not completely eliminated, light leaking has been significantly reduced. This makes HDDAGI more suitable for use in interiors and mixed outdoor-indoor scenes.

### Improved accuracy thanks to energy conservation

SDFGI had an issue with its spherical harmonics implementation, which turned out to be not energy-conserving. This made SDFGI look more saturated than "ground truth" renders would be, as can be seen here:

**SDFGI:**

![SDFGI vs HDDAGI reflection bounce feedback](/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_1.webp)

![SDFGI vs HDDAGI reflection bounce feedback](/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_2.webp)

**HDDAGI:**

![SDFGI vs HDDAGI reflection bounce feedback](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_1.webp)

![SDFGI vs HDDAGI reflection bounce feedback](/storage/blog/introducing-hddagi-real-time-global-illumination/hddagi_2.webp)

HDDAGI is energy-conserving, which also means its default **Bounce Feedback** value has been raised from `0.5` to `1.0` to compensate. This provides a more realistic appearance, especially for reflections whose brightness better matches the surrounding world:

**SDFGI with Bounce Feedback set to 0.5 (left), HDDAGI with Bounce Feedback set to 1.0 (right):**

![SDFGI vs HDDAGI reflection bounce feedback](/storage/blog/introducing-hddagi-real-time-global-illumination/sdfgi_vs_hddagi_reflection_bounce_feedback.webp)

## How to use HDDAGI

The process of using HDDAGI in your project is similar to SDFGI. Create some 3D nodes with geometry, ensure their GI mode is **Static** (as dynamic occluders are not supported yet), then enable **Dynamic GI** in the WorldEnvironment node (formerly **SDFGI**). No mesh preprocessing or baking is required.

In existing projects, HDDAGI will automatically replace SDFGI as the SDFGI code is no longer present in Godot 4.3.

## HDDAGI tradeoffs

To improve rendering performance, global illumination now runs in half-resolution by default. This change also affects VoxelGI, as the half-resolution GI project setting is used for both VoxelGI and HDDAGI (it also affected SDFGI). However, despite this resolution cut, the resulting quality is still generally greater than it was before. This is in part thanks to an improved upscaling algorithm used for HDDAGI.

If you have spare GPU headroom, you can disable the **Rendering > Global Illumination > GI > Use Half Resolution** project setting to make GI render at full resolution.

## Future plans

### Dynamic object support

Like SDFGI, HDDAGI currently only supports dynamic lights, not dynamic occluders (or dynamic emissive objects). Support for dynamic objects in HDDAGI is planned in a future release.

### Optional high density mode for higher-quality global illumination

While HDDAGI can provide good global illumination at a macro scale, the micro scale can sometimes leave to be desired due to the limited precision of voxel probes. Small objects could be better connected to the ground if there was a way to increase probe density where it matters most.

This would involve using raytracing to provide more accurate indirect lighting for smaller objects (smaller than the distance between two voxels).

## References

- [HDDAGI pull request](https://github.com/godotengine/godot/pull/86267)
- [Original HDDAGI pull request](https://github.com/godotengine/godot/pull/86007) (contains additional history)

## Support

If you would like to help with the development of these features, please, consider [supporting the project financially](https://fund.godotengine.org/)! More funding allows us to sponsor volunteer contributors and better respond to technical demands of project users.
