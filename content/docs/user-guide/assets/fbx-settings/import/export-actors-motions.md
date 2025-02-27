---
description: ' See the best practices for exporting skinned meshes for actors for
  the Animation Editor in Open 3D Engine. '
title: Exporting Actors and Motions
---

Use the following best practices when you export your skinned meshes using the **FBX Settings** tool.
+ If you use the z-up world coordinate system, use the following guidelines:
  + Ensure that your DCC scene is set to z for the up-axis world coordinate system.
  + Set the axis conversion up axis to z when exporting an `.fbx` file.
+ If you use the y-up world coordinate system, use the following guidelines:
  + Ensure that your DCC scene is set to y for the up-axis world coordinate system.
  + Set the axis conversion up axis to y when exporting an `.fbx` file.
+ When you create an `.actor` file, export your skinned mesh at the bind pose without any keyframes.
+ When you create a `.motion` file, bake animations before you export. Alternatively, bake animations when you use your DCC's `.fbx` export tools.
+ Export only the skeleton and mesh. Do not use transforms, groups, or parent nodes in the hierarchy above your root joint. The root joint must be the top parent of the skeletal hierarchy to ensure that motion extraction works properly.
+ Remove unused geometry, bones, vertices, materials, and nodes that are not necessary for the `.fbx` asset. This reduces the processing time and offers a better chance that the automatic processing works properly without making adjustments later. In your DCC, consider naming nodes with `_ignore` as a suffix to prevent those nodes from being processed.
+ Use the following guidelines for vertex count:
  + The theoretical maximum number of vertices for skinned meshes after processing is 4,294,967,295 (232 - 1). Although this limit is exceptionally high, we recommend that you follow best practices when modeling your skinned meshes. Use a reasonable polygon count that's suitable for the game device for which you are developing. Experiment with the number of polygons to get the desired quality while balancing game performance. You may need to adjust the polygon count if your game displays many actors at once. The value range for polygon counts can vary between 1,000 to 30,000. The polygon count that you should use depends on the game device, and the performance and quality of the actor.
  + The more vertices in an `.fbx` file, the longer Asset Processor takes to process it. View Asset Processor often and check for errors.

## Using the Maya Game Exporter 

The following are typical settings for the Maya Game Exporter when you export your skinned mesh character into O3DE.

**Export settings for .actor files**
+ Use the **Model** tab to export your `.fbx` files.

![Models tab in the Maya Game Exporter.](/images/user-guide/actor-animation/fbx-settings-actors-model-tab.png)
+ Use the following settings:
  + Select **Export Selection** from the drop-down list.
  + Select the **Skinning** check box.
  + If you are exporting blendshapes (morph targets), select the **Blendshapes** check box.
  + Clear the **Animation** check box. If this is selected, a `.motion` file is created.
  + Select your world coordinate system from the **Up Axis** drop-down list.
  + Navigate to a save path and specify the name of your `.fbx` file.
+ When you're done choosing these settings, select all of the bones for the character's skeleton and all of the skinned meshes.
+ Click **Export**.

**Export settings for .motion files (Animation Clips tab)**
+ Use the **Animation Clips** tab to export your `.fbx` files.

![Animation Clips tab in the Maya Game Exporter.](/images/user-guide/actor-animation/fbx-settings-motions-animation-clips-tab.png)
+ Use the following settings:
  + Select **Export Selection** from the drop-down list.
  + Click the **+** button to add an animation clip.
  + For **Clip Name**, enter a name for the clip.
  + For **Start** and **End**, specify a frame.
  + Select the **Bake Animation** check box.
  + Select your world coordinate system from the **Up Axis** drop-down list.
  + Navigate to a save path and specify the name of your `.fbx` file.
+ When you're done choosing these settings, select all of the bones for the character's skeleton.
+ If you are exporting blendshapes with your animation, also select the skinned mesh that has the blendshape animations. For example, if your character's face mesh has blendshapes, select the skeleton and the character's face mesh.
+ Click **Export**.

**Export Settings for .motion files (Time Editor tab)**
+ Use the **Time Editor** tab to export your `.fbx` files.

![Time Editor tab in the Maya Game Exporter.](/images/user-guide/actor-animation/fbx-settings-motions-time-editor-tab.png)
+ Use the following settings:
  + Select **Export Selection** from the drop-down list.
  + Select the clip that you want to export from the **Time Editor Clips** drop-down list.
  + Click the **+** button to add an animation clip.
  + Select the **Bake Animation** check box.
  + Select your world coordinate system from the **Up Axis** drop-down list.
  + Navigate to a save path and specify the name of your `.fbx` file.
+ When you're done choosing these settings, select all of the bones for the character's skeleton.
+ If you are exporting blendshapes with your animation, also select the skinned mesh that has the blendshape animations. For example, if your character's face mesh has blendshapes, select the skeleton and the character's face mesh.
+ Click **Export**.
