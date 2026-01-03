# Improving 3D Model Performance With LOD Control

In the construction industry, 3D models provide a level of visual context that simply cannot be matched through spreadsheets. However, working with them often requires specialized software and heavy-duty hardware. For quick mock-ups and visualizations sometimes all you need is one readily availabe tool- the web browser.

In this paper, I propose a proof of concept whereby 3D models can be hosted and shared on any webpage and viewed on any device with a web browser- no dedicated software needed. I explore the basics of 3D modelling, mesh compression techniques, creating scenes in `three.js`, and swapping between different versions of a mesh depending on how far away the user is from the object.

[The final scene](https://suryashch.github.io/3d_modelling/) contains 303 objects, each with a `low` and `hi` resolution version of their 3D mesh (red and green respectively), that dynamically render to the screen as the user zooms in.

![Working LOD Controlled Model](img/first-working-lod-model.gif)

Through the optimizations in this project, I was able to achieve an average `3x` improvement in GPU performance (peak `5x`, measured using number of `triangles`), while keeping draw calls constant. The [full code](https://github.com/suryashch/LOD-control-with-threeJS) and [research body of knowledge](https://github.com/suryashch/3d_modelling/blob/main/hosting-3d-model/per-object-lod-control-with-threejs.md) provide additional details to those interested.

## 3D Modelling Basics

The fundamental building blocks of 3D models are `vertices` and `edges`[^3]. `vertices` can be thought of as 'corners' while `edges` are what connect the corners to each other. In a cube, we have 8 `vertices` (in black) and 12 `edges` (in red), as seen in the image below (in no particular order).

![cubes edges and vertices defined](img/cubes-edges-vertices.png)

As the total number of `vertices` and `edges` in the scene increase, so does the strain on the GPU and as a result, it becomes laggy when you try to move around. Increasing the total number of `vertices` and `edges` in the scene also increases the workload for your computer.

A piece of geometry that is omnipresent in construction models is the pipe, modelled as a cylinder. However, circles don't have any corners- a circle can be thought of as infinitely many corners that all connect to each other. It is impossible to model a perfect circle, so it is approximated using a large number of `vertices`. The more `vertices`, the more the object starts looking like a circle.

Hence, when we extrude all these individual data points into the page, we get a cylinder (pipe), which is highly inefficient for how fundamental of a shape it is.

![Cylinder Vertices and Edges](img/cylinder.png)

Each one of these edges and vertices need to be kept track of by the computer's GPU, per pipe. **If not properly optimized, a single pipe in your model may contain as many data points as the entire building's structural frame.** Reducing the density of the pipe mesh is a quick way to improve the performance of our final `scene`.

## Reducing the Density of the Mesh

The density of a mesh is a measure of how many individual `vertices` and `edges` exist within it. The density of the mesh can be reduced by removing some `vertices` from it. By removing these data points however, we sacrifice on details and as can be seen in the image below- the finer details of the mesh are lost.

![Cylinder Comparison Before and After Compression](img/cylinder_decimate_comparison.png)

One easy way to reduce the mesh density is using a technique called `Decimate` [^3] in Blender [^13]. This modifier will remove `edges` in a mesh upto a specified `ratio`, while maintaining the overall shape of the object. The results below show a test case of `decimating` a 3D mesh model of a human foot [^1] up to a ratio of 0.1.

![Mesh model of human foot before and after compression](img/foot_comparison.png)

The mesh has lost a lot of visual quality. But, if we zoom out far enough-

![Mesh models of human foot zoomed out to show similarities](img/foot_comparison_zoomed.png)

Both meshes look similar. **At far enough distances, mesh quality can be reduced with limited change to visual context.** A `scene` that dynamically changes quality if the user is near or far may potentially impove performance.

## Loading a 3D Model to a Webpage

The JavaScript library `three.js` [^9] provides useful tools for viewing 3D models in a web-based environment. The basic concept behind `three.js` is to create a `scene` and add objects to it, like `lights`, `cameras`, `backgrounds`, and of course, `3D objects`[^5].

The file format we use for our 3D objects is the `GLTF` [^12] file format. `GLTF` is an open source 3D file format that is optimized for rendering in a web environment.

The 3D model being used for testing is that of a `Piperack` [^2]. The base file size is ~7MB; a relatively small 3D model. The model is loaded to the scene along with some lights, cameras, and spatial grid for reference.

![Scene with Model](img/background_with-model.png)

## Basic LOD in three.js

LOD (Level of Detail) modelling involves creating low and high resolution meshes for each object in the scene, and dynamically rendering each one based on how far the object is from the camera [^6]. This way, far-away objects can render in `low-res` and near objects can render in their full high definition. Since the total number of `vertices` and `edges` are less in the `low-res` model, switching to these meshes when they are a certain distance away will reduce the strain on the GPU.

In `three.js`, LOD control is done using the `three.LOD` class [^10]. At a high level, a `LOD` can be thought of as a container that holds meshes. Based on some distance threshold, the `LOD` swaps which mesh is active at any time. In this example, 3 versions of the `human foot` [^1] mesh are created- `hi-res`, `med-res`, and `low-res`, corresponding to 1, 0.4, and 0.1 `decimate ratios` respectively. The meshes have been coloured for identification purposes.

![Foot model LOD version colored](img/human-foot-LOD-versions-color.png)

These 3 meshes are loaded into one `LOD` container with distance thresholds of 10 units and 5 units from the camera. Now, zooming into the page changes the active mesh based on the specified distance thresholds [^7].

![Foot model LOD containers aligned and colors changed](img/foot-lod-pos-synced-color.gif)

Let's extend this knowledge out to our `Piperack` model [^2].

## Per Object LOD in three.js

In this section, we replicate our `LOD` results above to a larger 3D model with many more objects. The first step is to create a low-resolution version of the `Piperack` [^2] model at 0.4 `decimate ratio`. For easy identification, the meshes are color coded- `low-res` meshes in `red` and `hi-res` meshes in `green`. Both versions of the model are overlayed in the same file.

![Per Object LOD Control Color Coded](img/per-object-lod-control-mesh-colors.png)

Creating the LODs involves looping through the scene and for each object, saving the `low-res` and `hi-res` meshes to one `LOD` container. This process utilizes a function called `.traverse()` [^11]. Naming of the objects in the scene here is key, as the name enables the traversal function to identify the low and high resolution meshes.

A `Map()` object is used to store key-value pairs. Each key in the map contains the Object `name`, and data associated with the high and low resolution mesh. Conceptually, this is what the scene tree looks like after traversing the model.

![Working Scene Tree](img/scene-tree-working.png)

Each object in the scene is saved to one `LOD` container, and each container contains 2 meshes. The meshes are triggered to change at a distance of 5 units.

Now upon loading the `LOD` model to the scene, this is what we're greeted with.

![Fnal LOD Controlled Model](img/refined-lod-model.gif)

The initial load shows all objects rendered in `low-res` mode. Zooming in to specific objects causes them to render in `hi-res`. We measure the performance of the scene against 2 main metrics- `draw calls` (proxy for CPU usage) and `triangles` (proxy for GPU usage). Here are some key results.

![Performance Results 50ft View](img/performance-results-50ft-view.png)

At a high level 50ft view, only the `low-res` version of the model is active. Hence we observe our best performance improvements, at roughly 5x `triangles` reduction.

![Performance Results West End of Piperack](img/performance-results-westend-of-piperack.png)

The complex geometry of the wellhead in the background is rendered in `low-res` in the dynamic model, significantly reducing the number of `triangles` being tracked by the GPU.

![Performance Results of Main Piperack](img/performance-results-main-piperack.png)

The main Piperack of the scene is loaded dynamically- far away objects in `low-res` while near objects in `hi-res`. This will be on average, the compression capabilities that can be achieved in everyday use.

The full explanation and in-depth analysis of results can be found in this repo [^8].

## Conclusion

Out in the field, understanding information from drawings is key, and no spreadsheet can compete with the spatial context provided by 3D models. However, a true lightweight, cross platform solution for 3D visualization is lacking. Taking advantage of the web browser unlocks a readily availble tool whose output can be integrated into existing data pipelines and dashboards. With recent advances in cloud computing, what used to be a traditionally computation-heavy resource is starting to look more attainable for the masses.

In further research, I explore customizing the scene- changing colours, filtering for specific conditions, and running simulations. As well, I explore scaling up- identifying optimizations that can be made to further increase our file size, while limiting the effects on performance.

You can find more information about this research on my [github](https://github.com/suryashch/3d_modelling).

## Links

[^1]: human_foot_model https://sketchfab.com/3d-models/human-foot-base-mesh-b6dd50f7e87441dca79e24f8c702f84f

[^2]: piperacks_model https://free3d.com/3d-model/pipe-racks-building-blocks-bundle-2755.html?dd_referrer=

[^3]: analysis_decimate https://github.com/suryashch/3d_modelling/blob/main/reducing-mesh-density/analysis_decimate.md

[^5]: hosting_3d_model https://github.com/suryashch/3d_modelling/blob/main/hosting-3d-model/analysis_threejs.md

[^6]: superposing_models https://github.com/suryashch/3d_modelling/blob/main/hosting-3d-model/analysis_superposing-models.md

[^7]: basic_lod_control https://github.com/suryashch/3d_modelling/blob/main/hosting-3d-model/basic-lod-control-with-threejs.md

[^8]: per_obj_lod https://github.com/suryashch/3d_modelling/blob/main/hosting-3d-model/per-object-lod-control-with-threejs.md

[^9]: three.js https://threejs.org/docs/

[^10]: three_lod https://threejs.org/docs/#LOD

[^11]: three.object3d.traverse() https://threejs.org/docs/#Object3D.traverse

[^12]: gltf https://www.khronos.org/Gltf

[^13]: blender https://www.blender.org/
