Here is the English version of the README.md, ready to be copied and pasted into your GitHub repository.

📦 Blender PBR Auto-Importer (Cycles & Octane)
A powerful "One-Click" addon for Blender that automates the import of PBR texture sets (from ZIP files or folders). It handles not only the shader setup but also automatically imports the corresponding 3D model (FBX/OBJ/GLTF) if present.

Available in two distinct editions:

Cycles/Eevee Edition (Standard Blender Workflow)

Octane Render Edition (Specialized for Octane Nodes)

✨ Key Features
🚀 Automation
Direct ZIP Import: Select a .zip file — the addon extracts it, locates textures and models, and sets everything up instantly.

Auto-Mesh Detection: If the script finds a 3D file (.fbx, .obj, .gltf, .glb) inside the folder/zip, it imports it automatically.

LOD Support: Automatically prioritizes LOD0 files if multiple Level of Detail versions are found.

Decal / Plane Mode: If no mesh is found and nothing is selected, the addon creates a plane with the correct aspect ratio based on the texture dimensions (perfect for decals or walls).

🧠 Smart Shader Setup
The script uses a "Strict Matching" logic to assign textures to the correct sockets, preventing common errors (e.g., confusing Normal maps with Roughness maps).

Supported Channels:

Albedo / Diffuse / Base Color

Ambient Occlusion (AO) – automatically mixed with Albedo via Multiply

Roughness & Metallic

Normal Map (OpenGL/DirectX) & Bump

Transmission & Opacity/Alpha

Displacement & Height

Auto-Fixes:

Automatically sets Color Space to Non-Color for data maps (Roughness, Normal, Metallic).

Enables Hashed Alpha & Shadow Mode for transparent textures (Cycles/Eevee).

Sets up Mapping Nodes for global scaling.

🔧 Editions & Specifics
1. Cycles / Eevee Importer
Shader: Based on the Principled BSDF.

Compatibility: Optimized for Blender 4.2+ (works with 3.x, utilizes new 4.0+ sockets like "Transmission Weight" where available).

Displacement: Automatically enables "Displacement Only" in material settings.

2. Octane Render Importer
Shader: Uses the Universal Material.

Nodes: Dynamically locates the correct Octane nodes (Octane Image Tex, Octane Transform, etc.).

Requirement: Requires the Blender Octane Edition Plugin.

📥 Installation
Download the Python file you need:

cycles_pbr_loader.py for standard Blender users.

octane_pbr_importer.py for Octane users.

In Blender, go to Edit > Preferences > Add-ons.

Click Install... and select the file.

Enable the checkbox next to the addon name.

🎮 Usage
The addon integrates seamlessly into the File > Import menu but also provides hotkeys for a fast workflow.

Method A: Menu
Go to File > Import:

Cycles PBR (Auto ZIP) / (Auto Folder)

Octane PBR (Auto ZIP) / (Auto Folder)

Method B: Hotkey (Quick Import)
Default Hotkey: Shift + W (in 3D View).

This triggers the file browser for the ZIP Import mode.

📝 License & Credits
Developed by 680pxl. Free to use and modify.
