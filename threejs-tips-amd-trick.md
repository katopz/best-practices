<div class="nav-wrapper fixed"><input type="checkbox" id="nav-panel-left-trigger" class="nav-panel-trigger"> <label for="nav-panel-left-trigger" style="visibility:hidden" class="button-menu small-screen-only nav-panel-left-trigger-label show-always">Contents</label> <input type="checkbox" id="nav-panel-right-trigger" class="nav-panel-trigger"> <label for="nav-panel-right-trigger" class="button-menu small-screen-only nav-panel-right-trigger-label show-always">Menu</label>

<div class="nav-panel-triggers-bg">

<div class="top-nav-pagination"><a class="top-nav-pagination-prev disabled"></a><a class="top-nav-pagination-next disabled"></a></div>

</div>

<nav class="nav-panel nav-right ">

## [Discover three.js](/)

*   [Table Of Contents](/book/contents/)
*   [Live Code Examples](/examples/)
*   [Apps](/apps/)
*   [Tips and Tricks](/tips-and-tricks/)
*   [Stay up to date - sign up to the mailing list!](/#mailing-list-signup)

</nav>

[<span class="fas fa-arrow-up fill"></span>](#top "Scroll to top")

<div class="nav-panel nav-left">

<nav class="toc">

### [Contents](//)

*   <div><span class="fa fa-arrow-right toc-active-icon" aria-hidden="true"></span><span>The Big List of three.js Tips and Tricks!</span><span class="chapter"></span></div>

</nav>

</div>

</div>

# THE BIG LIST OF three.js TIPS AND TRICKS!

Hey everyone! While writing the book I’ve been gathering lots of tips, tricks, caveats, and gotchas. Here’s a bit list of everything that I’ve found so far, and I’ll be updating it from time to time so check back here occasionally.

Not all of the tips here have been experimentally verified, especially the performance tips - there are too many variables involved to blindly follow a list, so always make sure to test your own app thoroughly and see what works for you. These are suggestions, not rules (mostly). That said, this page should make a good starting point for apps of any size.

If you have anything to add or notice any mistakes, let me know on your preferred communication medium. There are lots of contact links in the footer.

Most of the tips here are not specific to three.js, or even WebGL, but apply in many real-time graphics applications.

Happy Coding!

## Beginner Friendly Tips, or _Help! Why Can’t I See Anything?_

You’ve followed a couple of basic tutorials and everything worked fine, but now you’re creating your own app and you’ve set everything up _exactly_ as the tutorial says. But you just can’t see anything! WTH??

Here’s a couple of things you can do to figure out the problem.

### 1\. Check the [browser console](https://developer.mozilla.org/en-US/docs/Tools/Browser_Console) for error messages

But you already did that, right?

### 2\. Set the background color to something other than black

Staring at a black canvas? It’s hard to tell whether something is happening or not if all you can see is black. Try setting the background color to red:

<div class="highlight">

    scene.background = new THREE.Color( 'red' );

</div>

If you get a red canvas, that’s a means that your `renderer.render` calls are working, and you can move on to figuring out what else is wrong.

### 3\. Make sure that you have light in your scene and that it’s illuminating your objects

Just like in the real world, most materials in three.js need light to be seen. One exception is the [`MeshBasicMaterial`](https://threejs.org/docs/#api/en/materials/MeshBasicMaterial), so you can temporarily override all the materials in your scene to see whether this is your problem.

<div class="highlight">

    scene.overrideMaterial = new THREE.MeshBasicMaterial( { color: 'green' } );

</div>

### 4\. Is your object is within the camera’s viewing frustum?

If your object is not inside the [viewing frustum](/book/first-steps/first-scene/#viewing-frustum), it will get clipped. Try making your far clipping plane really big:

<div class="highlight">

    camera.far = 100000;
    camera.updateProjectionMatrix();

</div>

Remember this is just for testing though! The camera’s frustum is measured in meters, and you should make it as small as possible for best performance.

### 5\. Is your camera inside the object?

By default, everything gets created at the point $(0,0,0)$, AKA the **origin**. Make sure you have moved your camera back so that you can see your scene!

<div class="highlight">

    camera.position.z = 10;

</div>

### 6\. Think carefully about the scale of your scene

Try to visualize your scene, and remember that one unit in three.js is one meter. Does everything fit together in a reasonably logical manner?

## General Tips

1.  Object creation in JavaScript is expensive, so don’t create objects in a loop. Instead, create a single object such as a [Vector3](https://threejs.org/docs/#api/en/math/Vector3) and use [`vector.set()`](https://threejs.org/docs/#api/en/math/Vector3.set) or similar methods inside your loop
2.  The same goes for your render loop - make sure you are doing as little work as possible in your render loop since you want it to run smoothly 60 times per second
3.  Always use [`BufferGeometry`](https://threejs.org/docs/#api/en/core/BufferGeometry) instead of [`Geometry`](https://threejs.org/docs/#api/en/core/Geometry), it’s faster
4.  The same goes for the pre-built objects, always use the buffer geometry version ([`BoxBufferGeometry`](https://threejs.org/docs/#api/en/geometries/BoxBufferGeometry) rather than [`BoxGeometry`](https://threejs.org/docs/#api/en/geometries/BoxGeometry))
5.  Always try to reuse objects such as objects, materials, textures etc (bearing in mind updating some things may be slow, see texture tips below)

## Work in SI Units

three.js is developed using SI units. If you stick with this convention you will find things easier, and if you break it then make sure that you do so for a good reason!

*   Distance is measured in **meters** (1 three.js unit = 1 meter)
*   Time is measured in seconds
*   Light is measured in SI light units, [Candela](http://www.si-units-explained.info/luminosity/) (cd), Lumen (lm), and Lux (lx)

If you creating things on a truly epic scale (space simulations and things like that), either use a scaling factor or switch to using a [logarithmic depth buffer](http://threejs.org/examples/#webgl_camera_logarithmicdepthbuffer).

## Accurate Colors

For (nearly) accurate colors, use these settings for the renderer:

<div class="highlight">

    renderer.gammaFactor = 2.2;
    renderer.gammaOutput = true;

</div>

Then for colors do this:

<div class="highlight">

    const color = new Color( 0x800080 );
    color.convertSRGBToLinear();

</div>

Or, in the more common case of using a color in a material:

<div class="highlight">

    const material = new MeshBasicMaterial( { color: 0x800080 } );
    material.color.convertSRGBToLinear();

</div>

Finally, to get (nearly) correct colors in your textures, you’ll need to set the texture encoding for the color, environment, and emissive maps _only_:

<div class="highlight">

    const colorMap = new TextureLoader().load( 'colorMap.jpg' );
    colorMap.encoding = sRGBEncoding;

</div>

All other map types should remain in linear color space, so don’t change them!

Note that I’m saying **nearly correct** here since three.js color management is actually not correct at the moment. Hopefully, it will be fixed soon, but in the meantime, the difference in color will be so minor that it’s very unlikely anybody will notice unless you are doing scientific or medical renderings.

## JavaScript

The JavaScript engines used by web browsers change frequently and do an amazing amount of optimization of your code behind the scenes. Don’t trust your intuition about what will be faster, always test. Don’t trust articles from a few years ago telling you to avoid certain methods such as `array.map` or `array.forEach`. Test it for yourself.

## Models, Meshes and Other Visible Thing

1.  Avoid using common text-based 3D data formats, such as Wavefront OBJ or COLLADA, for asset delivery. Instead, use formats optimized for the web, such as glTF
2.  use Draco mesh compression with glTF
3.  If you need to make large groups of objects visible and invisible (or add/remove them from your scene), consider using [Layers](https://threejs.org/docs/#api/en/core/Layers) for best performance
4.  Objects at the same exact same position cause flickering. Try offsetting things by a tiny amount like $0.001$ to make things look like they are in the same position, but keep your GPU happy
5.  Keep your scene centered around the origin to prevent floating point errors at large coordinates
6.  Never move your Scene object. It gets created at $(0,0,0)$, and this is the default frame of reference for all the objects inside it.

## Camera

1.  Make your frustum as small as possible for better performance. It’s fine to use a large frustum in development, but once you are fine-tuning your app for deployment, make your frustum as small as possible to gain a few FPS
2.  Don’t put things right on the far clipping plane (especially if your far clipping plane is really big), this can cause flickering

## Renderer

1.  Don’t enable [`preserveDrawingBuffer`](https://threejs.org/docs/#api/en/renderers/WebGLRenderer) unless you need it
2.  Disable the alpha buffer unless you need it
3.  Don’t enable the stencil buffer unless you need it
4.  Disable the depth buffer unless you need it (but you probably do need it)
5.  use `powerPreference: "high-performance"` when creating renderer. This _may_ make a users system choose the high-performance GPU, in multi-GPU systems.
6.  Consider only rendering when the camera position changes by epsilon or when an animation happens
7.  If your scene is static and uses `OrbitControls`, you can listen for the `change` event to only render when the camera moves:

<div class="highlight">

    OrbitControls.addEventListener( 'change', () => renderer.render( scene, camera ) );

</div>

You won’t get a higher frame rate from the last two, but what you will get is fewer laptop fans switching on, and less battery drain on mobile devices.

Note: I’ve seen a few places around the web recommending that you disable antialiasing and apply a post-processing AA pass instead. In my testing, this is not true. On modern hardware built-in MSAA seems to be extremely cheap even on low-power mobile devices, while the post-processing FXAA or SMAA passes cause a considerable frame drop

## Lights

1.  Lights, especially `SpotLight`, `PointLight`, `DirectionalLight` are slow. Use as few lights as possible in your scenes
2.  Avoid adding and removing lights from your scene, since this requires the WebGLRenderer to recompile all shader programs (it does cache the programs so subsequent times that you do this it will be faster than the first)
3.  Turn on `renderer.physicallyCorrectLights` for accurate lighting that uses the SI units

## Shadows

1.  If your scene is static, only update the shadow map when something changes, rather than every frame
2.  Use a [`CameraHelper`](https://threejs.org/docs/#api/en/helpers/CameraHelper) to visualize the shadow camera’s viewing frustum
3.  Remember that point light shadows are more expensive than other shadow types since they must render six times (once in each direction), compared with a single time for `DirectionalLight` and `SpotLight` shadows
4.  While we’re on the topic of `PointLight` shadows, note that the `CameraHelper` only visualizes _one_ out of _six_ of the shadow directions when used to visualize point light shadows. It’s still useful, but you’ll need to use your imagination for the other 5 directions

## Materials

The built-in three.js materials have a simple performance/quality tradeoff:

1.  `MeshStandardMaterial` highest quality/slowest
2.  `MeshPhongMaterial`
3.  `MeshLambertMaterial`
4.  `MeshBasicMaterial` lowest quality/fastest

Use the best quality material you can afford, and switch to lower quality materials when you need to.

1.  `MeshLambertMaterial` doesn’t work for shiny materials, but for matte materials like cloth it will give very similar results to `MeshPhongMaterial` but is faster
2.  If you are using morph targets, make sure you set [`morphTargets = true`](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial.morphTargets) in your material, or they won’t work!
3.  Same goes for [morph normals](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial.morphNormals)
4.  And if you’re using a [SkinnedMesh](https://threejs.org/docs/#api/en/objects/SkinnedMesh) for skeletal animations, make sure that [`material.skinning = true`](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial.skinning)
5.  Materials used with morph targets, morph normals, or skinning can’t be shared. You’ll need to create a unique material for each skinned or morphed mesh ([`material.clone()`](https://threejs.org/docs/#api/en/materials/Material.clone) is your friend here).

## Custom Materials

1.  Only update your uniforms when they change, not every frame.

## Geometry

1.  Avoid use of line loop since it must be emulated by lines strip

## Textures

1.  All of your textures need to be power of two (POT) size: $1, 2, 4, 8, 16, …, 512, 2048, …$
2.  Don’t change the dimensions of your textures. Create new ones instead, [it’s faster](https://webglinsights.github.io/tips.html)
3.  Use the smallest texture sizes possible (can you get away with a 256x256 tiled texture? You might be surprised!)
4.  Non-power-of-two (NPOT) textures require linear or nearest filtering, and clamp-to-border or clamp-to-edge wrapping. Mipmap filtering and repeat wrapping are not supported. But seriously, just don’t use NPOT textures
5.  All textures with the same dimensions are the same size in memory, so JPG may have a smaller file size than PNG, but it will take up the same amount of memory on your GPU

## Post-Processing

1.  The built-in antialiasing doesn’t work with post-processing (at least in WebGL 1). You will need to do this manually, using [FXAA](https://threejs.org/examples/#webgl_postprocessing_fxaa) or [SMAA](https://threejs.org/examples/#webgl_postprocessing_smaa) (probably faster, better)
2.  Since you are not using the built-in AA, be sure to disable it!
3.  three.js has loads of post-processing shaders, and that’s just great! But remember that each pass requires rendering your entire scene. Once you’re done testing, consider whether you can combine some of your passes into one single custom pass

## Disposing of Things

Removing something from your scene?

First of all, consider not doing that, especially if you will add it back again later. You can hide objects temporarily using `object.visible = false` (works for lights too), or `material.opacity = 0`. You can set `light.intensity = 0` to disable a light without causing shaders to recompile.

If you do need to remove things from your scene _permanently_, read this article first: [How to dispose of objects](https://threejs.org/docs/#manual/en/introduction/How-to-dispose-of-objects)

## Updating Objects in Your Scene?

Read this article: [How to update things](https://threejs.org/docs/#manual/en/introduction/How-to-update-things)

## Performance

1.  set `object.matrixAutoUpdate = false` for static or rarely moving objects and manually call `object.updateMatrix()` when their position/quaternion/scale are updated
2.  Transparent objects are slow, use as few transparent objects as possible in your scenes
3.  use [`alphatest`](https://threejs.org/docs/#api/en/materials/Material.alphaTest) instead of standard transparency if possible, it’s faster
4.  When testing the performance of your apps, one of the first things you’ll need to do is check whether it is CPU bound, or GPU bound. Replace all materials with basic material using `scene.overrideMaterial` (see beginners tips and the start of the page). If performance increases, then your app is GPU bound.
5.  When performance testing on a fast machine, you’ll probably be getting the maximum frame rate of 60fps. Run chrome using `open -a "Google Chrome" --args --disable-gpu-vsync` fo ran unlimited frame rate
6.  Modern mobile devices have high pixel ratios as high as $5$ - consider limiting max pixel ratio to 2 or 3 on these devices at the expense of some very slight blurring of your scene
7.  Bake lighting and shadow maps to reduce the number of lights in your scene
8.  If you have the resources and time, use compressed textures. Unfortunately setting these up for the web is a pain at the moment since there is no one format supported by all devices
9.  Keep an eye on the number of drawcalls in your scene. A good rule of thumb is fewer draw calls = better performance
10.  Far away objects don’t need the same level of detail as objects close to the camera. There are many tricks used to increase performance by reducing the quality of distant object. For example, the [LOD](https://threejs.org/docs/#api/en/objects/LOD) (Level Of Detail) object stores different objects for different distance (for example, near, medium, and far). You may also get away with only updating position / animation every 2nd or 3rd frame for distant objects

## Advanced Tips

1.  Don’t use `TriangleFanDrawMode`, it’s slow.
2.  use geometry instancing when you have hundreds or thousands of similar geometries
3.  Animate on the GPU instead of the CPU, especially when animating vertices or particles (see [THREE.Bas](https://github.com/zadvorsky/three.bas) for one approach to doing this)

## Read These Pages Too!

Unity and Unreal engine also have pages with lots of performance suggestions, most of which are equally relevant for three.js. Make sure to read over them as well.

*   [Optimizing graphics performance (Unity)](https://docs.unity3d.com/Manual/OptimizingGraphicsPerformance.html)
*   [Performance Guidelines for Artists and Designers (Unreal)](https://docs.unrealengine.com/en-us/Engine/Performance/Guidelines)

WebGL Insights has a lost of tips collected from all their chapters. It’s more technical, but worth reading too, especially if you are writing your own shaders.

*   [WebGL Insights Tips](https://webglinsights.github.io/tips.html)

## References

*   [@jackrugile and @mrdoob on Twitter](https://mobile.twitter.com/jackrugile/status/966440290885156864)
*   [A-Painter performance optimizations](https://blog.mozvr.com/a-painter-performance-optimizations)
*   [twitter](https://twitter.com/lewy_blue/ "My Twitter Profile")
*   [discourse](https://discourse.threejs.org/u/looeee/ "The three.js Forum")
*   [linkedin](https://www.linkedin.com/in/lewy-blue-30b9b193/ "My LinkedIn Profile")
*   [github](https://github.com/looeee/ "My GitHub Profile")
*   [instagram](https://www.instagram.com/discover_threejs/ "Discover three.js on Instagram")
*   [facebook](https://www.facebook.com/discoverthreejs "Discover three.js on Facebook")
*   [© DISCOVER three.js](https://discoverthreejs.com/)
