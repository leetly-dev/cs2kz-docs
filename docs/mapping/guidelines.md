# Mapping Guidelines

The aim of this document is to help guide new and experienced mappers towards improving the quality of their CS2KZ maps in preparation for global map approval. Following this guide will provide insight and resolutions to issues that may prevent your map from being approved. This document is written under the assumption that you have previous experience using the Hammer map editor.

## Fundamentals Changes

In Source 2, the world and its objects are mesh based. This differs from Source 1's brush system whereby the world is built with 'blocks'. The new mesh based system offers a far more intuitive approach to building your map however will require some background research if you're coming from Source 1. It is highly recommended that you move towards this new system if you haven't already since many optimisation issues previously manageable within Source 1 will not work the same way and can lead to later visual issues under Source 2's mesh based system.

## Useful software/sites

- [Source2Viewer](https://source2viewer.com/), used for decompiling CSGO and CS2 assets.
- [source2.wiki](https://www.source2.wiki/CommunityGuides/installS2Sdk?game=cs2), wiki
- [developer.valvesoftware.com](https://developer.valvesoftware.com/wiki/Source_2/Docs), wiki
- [Text .vmat generator](https://jakkekz.github.io/fuk-point_worldtext/), made by jakke

## Lighting

Lighting helps to build an atmospheric and immersive world, it also plays a major role in the readability of your map. While the context changes from map to map, generally speaking this means that your lighting should strike a balance between being visually appealing and readable for gameplay. 

light_environment for sunlight will in most cases provide adequate lighting, however it's worth trying different angles, brightness and colour for the light to see what looks the best.

In shaded areas of your map, it may be necessary to incorporate a secondary light source to provide better visibility, however in doing so, it is recommended that the light is complimented by a source such as a candle or light bulb prop.

<div style="display: flex; gap: 5px;">
  <div style="flex: 1;">
    <img src="/badlight.jpg" alt="Bad lightning" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>1. Bad lighting</em></p>
  </div>
  <div style="flex: 1;">
    <img src="/goodlight.jpg" alt="Good lightning" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>2. Good lighting</em></p>
  </div>
</div>

Adjusting the luminosity/brightness/fade of your lights will help to make your lights more convincing. Only use light ranges higher than 1024 units sparingly as this can negatively impact fps and compile times.


### Lightmap Resolution and Lightmap Space
The lightmap resolution of your map can have a significant impact on the outcome of your map’s lighting. Best practice is to always do a final compile for your map which by default is set to 2048 resolution. Below this resolution you are more likely to encounter lighting artifacts, especially in darker areas of the map. In larger maps with more light space it may be necessary to compile your map at a lightmap resolution higher than 2048.

<div style="text-align: center;">
  <img src="/lightartifacting.jpg" alt="Artifacting" style="max-width: 600px; display: block; margin: 0 auto;">
  <p style="margin: 10px 0;"><em>3. Lighting artifacts due to low lightmap resolution</em></p>
</div>

With this in mind you also want to avoid calculating lighting for areas of the map never visible to the player. The first step in achieving this is to cull lightmap space from your map by deleting unnecessary faces. Any face that will never be visible to the player should be deleted (assuming it is not contributing to vis). The second step is to incorporate a lightmap playerspace mesh. This is a brush material which tells the engine where to calculate high resolution lightmap space and will tell the engine where lightmap space should be prioritised.

[Counter-Strike 2 Hammer - Basic Map Optimisations (compile time)](https://youtu.be/VGxPXnGJ0wM?si=XKQLfUU9U4Ijs_fJ&t=135), by ReDMooNTV

### Light probes and cubemaps
Light probes and cubemaps are required for your map and can be incorporated simultaneously with the env_combined_light_probe_volume entity. Light probes are required to create diffuse lighting on entities which cannot utilise direct lighting such as prop_dynamic entities. Cubemaps are required to create proper reflections for material surfaces and your view model. This is achieved by projecting a three dimensional image of a room onto the surfaces within the bounds of the entity.

<div style="display: flex; gap: 15px;">
  <div style="flex: 1;">
    <img src="/noprobe.jpg" alt="Present light probe" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>4. Combined light probe absent</em></p>
  </div>
  <div style="flex: 1;">
    <img src="/almostprobe.jpg" alt="Absent light probe" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>5. Combined light probe present, but improper edge fade</em></p>
  </div> 
  <div style="flex: 1;">
    <img src="/yesprobe.jpg" alt="Absent light probe" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>6. Combined light probe present</em></p>
  </div>
</div>

To properly implement an env_combined_light_probe_volume, the entity origin should be placed at the center of a room and at the head height of the player. If the room has multiple elevations, the entity origin can be placed in the middle. The entity origin can be moved with the pivot manipulation tool. If an object is obstructing the light probe, ensure that the origin is not placed within or halfway through that object. After finding a suitable location for the origin of the entity the bounds should be extruded to encapsulate the entirety of the room.

Visible seams occur where light probe volumes meet. To hide these, use an edge fade value of 8 or 16 for most areas, or 32 for larger transitions. When applying a fade, ensure the volume edges overlap by twice the fade distance to maintain a smooth blend.

When placing combined light probes near walls, floors, or ceilings, extend the volume so the edge fade overlaps the surface. If the fade ends exactly at the wall, the lighting influence drops to zero, and the surface won't receive proper reflections or bounce light as seen in image 5.

<div style="text-align: center;">
  <img src="/lightprobe.png" alt="Artifacting" style="max-width: 600px; display: block; margin: 0 auto;">
  <p style="margin: 10px 0;"><em>7. Combined light probe encapsulating the room with edge fade</em></p>
</div>

<div style="display: flex; gap: 15px;">
  <div style="flex: 1;">
    <img src="/nofade.gif" alt="Fade" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>8. Combined light probe without fade or overlap</em></p>
  </div>
  <div style="flex: 1;">
    <img src="/yesfade.gif" alt="No fade" style="width: 100%; display: block;">
    <p style="text-align: center; margin: 10px 0;"><em>9. Combined light probe with fade and overlap</em></p>
  </div> 
</div>

Irregularly shaped rooms often force light probe volumes to overlap in awkward areas, such as through walls into adjacent spaces. This creates messy transitions where surfaces sample the wrong light probes. To fix this, use the priority system to 'force' the correct probe to take over in those overlap zones. Usually, you’ll want the probe that best fits the specific room's shape or lighting to have the higher priority.

<div style="text-align: center;">
  <img src="/probepuzzle.png" alt="Probepuzzle" style="max-width: 600px; display: block; margin: 0 auto;">
  <p style="margin: 10px 0;"><em>10. A combined light probe "puzzle" with priorities mentioned</em></p>
</div>

<div style="text-align: center;">
  <img src="/fadeprio.png" alt="FadeAndPriority" style="max-width: 600px; display: block; margin: 0 auto;">
  <p style="margin: 10px 0;"><em>11. Combined light probe with fade and priority</em></p>
</div> 

## Texturing

Similarly to lighting, the textures you choose can create the atmosphere of your map and also impact the gameplay. 

### Custom Texturing and Materials
Custom textures and materials can be added to your addon using the material editor. To add a new texture/material you will need to create a new .vmat file. To create a .vmat file simply open the material editor, press new and save the file within “csgo_addons/your_addon/materials/”. You must save the file before you can edit it! 

The list of properties to choose from will vary depending on the shader type you choose. If you wish to change the shader you can do so and re-save that vmat without having to create a new one.

Opening the variables tab allows you to change the interactive properties of the material including the sound of footsteps and bullet impacts. Select material type and search through the list of available surfaces until you find an appropriate choice.

> [!NOTE]
Keep in mind that materials update in real time in game so it's easy to have these side by side to see how it changes.

### Useful shaders
Material shaders are different customizable configurations for materials each with varying properties and functions. Here we have listed some of the more useful ones with tips on how to use them.


- `Csgo Environment` shader

This shader allows **PBR** layering for generic materials such as wall, floor and terrain surfaces.

Although only a colour/texture file is required for the shader to function, you should aim to generate normal and roughness maps at a minimum to provide your materials with realistic topography and surface reflectivity.

Coming from Source 1 you may have used the lightmapped generic shader for your generic materials. While this shader is still available in Source 2, it should only be used sparingly as it lacks many of the features and fidelity offered by alternative shaders.

If you’re planning to use a material texture which lacks other PBR layers, you can generate them yourself. For those who want fast results you can quickly generate these layers from a colour texture using [NormalMap Online](https://cpetry.github.io/NormalMap-Online/) or [materialize](https://github.com/maikramer/Materialize).

You can also generate the layers manually using an appropriate photo editing software such as Gimp, Photoshop or Blender.

- `Csgo Environment Blend` shader

The environment blend shader allows you to combine two materials and create smooth transitions between those materials with the texture paint tool.

> [!WARNING]
> Explain how it is used in Hammer.

- `Csgo Water Fancy` shader

If you don’t plan on adding water to your map you can ignore this shader.
If you do, it is imperative that you create a custom water material rather than using the stock assets.

The easiest way to prepare a water material using this shader is to decompile a water material already available in the game. This will allow you to see how the shader properly functions.

Most people’s first issue when using stock water is that the surface starts to fade the further it is located from the world origin. To adjust the fade distance, change the UV max and minimum values.


- `Csgo Complex` shader

The Csgo Complex shader offers experimental properties for your materials such as emissive lighting, transparency, animation and movement.

To create Neon/Glowing/Illuminated materials with the “Csgo Complex” shader tick the “Self Illum” box. Scroll down and set the “Self Illum Mask” to white. Play around with the “Brightness” and “Albedo Factor” slider.

With glowing materials it's important to remember that it will cause light artifacting VERY easily if the light is mainly coming from Self Illum materials. Thats why it's recommended to disable "Emissive Lighting" under the mesh properties. Emissive in this case means that it reflects light from itself onto other things (it's bad)

To make the glowing material appear as “glowing” (without as many problems) a combination of Post Processing (with bloom) and the use of light entities is recommended. Good light entities for this case could be “light_rect” and “light_omni2” with either a sphere or one of the tubes as a light shape. Remember that less light entities are better than many.

Transparent materials are also done with the Csgo Complex shader. Just tick the “Translucent” box and configure it. For this you will need an alpha mask (ZONE GRADIANT)  (example image).

- `Csgo Static Overlay` shader

Can be used to project materials onto faces. The best use case for this would be images/art/lj numbers. This shader does also have the translucent option. 

Why does this differ from just making a face with Csgo Complex? 

Well because faces tend to glitch out when viewed from further away, this doesn’t happen with overlays, as seen in the gif below.

<div style="text-align: center;">
  <img src="/overlayface.gif" alt="OverlayVsFace" style="max-width: 600px; display: block; margin: 0 auto;">
  <p style="margin: 10px 0;"><em>12. Overlay and face comparison</em></p>
</div>

- `Sky` shader

Used for making custom skybox material. The “Sky Texture” can be added in multiple different forms. The preferred and best looking one is as an .exr file with HDR. This will however use the most file size.

The Dxt1 (LDR) option can be used if the skybox image isn't HDR. This option uses less file size. Use this option if your skybox is a simple .png image.
EXPLAIN WHAT CUBEMAP TEXTURE IS AND HOW TO GET/MAKE THEM

- `Csgo Moondome` shader

Moondomes are used because toolsskybox doesn’t work properly and renders things behind it. Moondomes are basically a material shader that acts like a proper skybox material. Keep in mind that it has collision on by default but this can be fixed by changing the “Surface Property” to “Default Silent” from “Default” in the “Attributes” tab. This makes the moondome act like a clip by not leaving gunshot dents but still acting like a wall that can’t be passed.

When making moondomes it’s important to keep in mind that the “Color” setting is set to a grey colour which should be made white to not have the moondome darker than the skybox. Then choose the skybox “Cube Map” texture and you're done.

- `Refract` shader

It's broken as this is being written. Has to be rendered as a model through a “info_particle_system” entity. 

Needs “FidelityFX Super Resolution” video setting disabled to show without visual bugs.

- `Csgo Weapon` shader

Can be used to make very interesting materials by enabling the SFX options “Glitter” and “Iridescence”. This shader does in fact work on other surfaces than weapons.

- `Csgo Simple 3layer Parallax` shader

Can be used to make fake rooms and layered materials such as ice. Check out [this](https://www.youtube.com/watch?v=6ZgSIaJoe8g) video to learn more. 

- `Csgo Composite Generic` shader

Used for loading screen images. The option A is the only one needed.

- `Csgo Lightmappedgeneric` shader

Do not use this, it doesn't work properly.


## Design/Detailing

### Infinite water
Water setup has changed significantly since CS:GO, largely due to how it interacts with light probes. To get it looking right, keep these points in mind:

- Larger Water Surface: The water area needs to be much larger than in CS:GO. This hides "bad" reflections near the horizon and gives the water’s edge fade more distance to blend out smoothly.

- Light Probe Coverage: Depending on your layout, a single large light probe is usually best. Never let light probe edges sit on top of the water, as the seams are incredibly obvious on reflective surfaces such as water.

- The 20,000 Unit Limit: Always center your water origin on the grid. Water will stop rendering entirely once it reaches 20,000 units from the origin on any side.

- 3D Skybox Transitions: Since the main map renders over the 3D skybox, ensure your main map geometry does not sit within the water's fade range. This prevents the "seam" between the real world and the skybox from becoming visible.

- If your map has a low perspective and players cannot look directly down onto the water, you can get by with only water fade.

- If the reflections from your env_combined_light_probe_volume look messy, identify the specific objects causing the issue. You can disable "Render to Cubemaps" on those individual entities to prevent them from being captured in the reflection, which often cleans up the final look.

### Diving deeper

If your map is tall (for example kz_avalon) the edge between the main map and 3d skybox is going to be noticeable, mostly due to lightprobes and fade.

Workarounds for this includes:
- Replacing the lightprobes “cubemap texture” with the one from the 3d skybox. 
- In the 3d skybox map right click your lightprobe and “Write Custom Cubemap…”, then select this .vtex file in the main map's lightprobe.
If this is done, remember to set this lightprobes priority lower than other, otherwise other lightprobes on the map will bug out.
Lightprobe configuration
Lightprobe “ball” should be roughly on the same height and placement in both of the maps (3d skybox and main). You can move the ball with the “Pivot Manipulation tool”
The lightprobe should be larger than the playable area (for example the skybox in the kz_avalon example.)
Water Configuration
Water can only be configured on new/decompiled water materials.
SSR
Right now SSR might be the best solution for reflections on these maps.
Downsides include:
Reflects what the user sees (no undersides or backsides for example)
Things moving off the camera will flicker and get distorted
SSR has a lot of configurable values in the material editor after being enabled.
Water Fade
The idea of water fade is to help with the transition between the main map’s water and the 3d skybox’ water.
These sliders configure the waters fade https://i.imgur.com/Ex63NWr.png
Other settings such as reflectance and glossiness can also help with the transition but it makes the water uglier.
Fog can help cover the fade but adds a circle of fog which is quite noticeable



Word’s workshop map https://steamcommunity.com/sharedfiles/filedetails/?id=3555595971
https://www.mediafire.com/file/n8mh5ddw7gmehyp/infinitewater.zip/file


		
### Particles
Enabling particle editor
https://developer.valvesoftware.com/wiki/Counter-Strike_2_Workshop_Tools/Particles
### Sounds
Encoding txt https://www.source2.wiki/CommunityGuides/encodingtxt?game=any
Wav mp3
Loop (wav looping with for example wavosaur) youtube links
