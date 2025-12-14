1. Prerequisite: Octane Render is Mandatory
The most important requirement is that Octane Render must be installed and active in Blender.

Why: The script specifically searches for Octane nodes (e.g., Universal Material, Octane Image Texture, Texture Displacement).

Result: If you try to run this in standard Blender (Cycles/Eevee), the script will fail or error out because it cannot find the necessary node types defined in find_octane_node_id.

2. File Naming Conventions (Crucial)
The addon uses a "Strict Matching" system to decide which image goes into which slot. For the auto-detection to work, your texture files must contain specific keywords in their filenames:

Albedo/Color: Must contain albedo, diffuse, or color.

Roughness: Must contain roughness or rough.

Normal: Must contain normal.

Metallic: Must contain metallic or metalness.

Displacement: Must contain height or disp.

Emission/Opacity: Supports trans, opacity, or alpha.

Important: The script ignores files containing words like "preview", "thumb", or "billboard".

3. Understanding the "Target Priority"
The script decides where to apply the material based on a specific hierarchy. You need to know this to control the outcome:

Priority 1: Mesh inside the Folder/ZIP:

If the folder (or ZIP) contains a 3D model (.fbx, .obj, .gltf), the script will import that model and apply the material to it automatically.

Note: It looks for "LOD0" specifically if multiple LODs are present.

Priority 2: Selected Object:

If no 3D model is found in the folder, the script looks at what you have selected in the Viewport. It will apply the material to your current selection.

Priority 3: Fallback Plane (Decal):

If no model is in the folder AND nothing is selected in the viewport, the script creates a new Plane automatically.

It scales this plane to match the aspect ratio of the texture (useful for decals or posters).

4. Texture Structure
One Map per File: The script is designed for "Discrete" workflows (one file for Roughness, one for Albedo, etc.). It does not appear to support "Packed" maps (e.g., Occlusion/Roughness/Metallic packed into R, G, and B channels of a single image). It loads the whole image into the node.

Supported Formats: Ensure textures are .jpg, .png, .exr, .tif, or .tiff.

5. Controls
Hotkey: The default shortcut is Shift + W.

Menu: You can find it in File > Import > Octane PBR....
