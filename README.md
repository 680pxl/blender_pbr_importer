# 📦 Blender PBR Auto-Importer (Cycles & Octane)

A powerful **one-click addon for Blender** that automates the import of PBR texture sets from ZIP files or folders.
It not only sets up shaders automatically but also imports the corresponding 3D model (FBX / OBJ / GLTF) if present.

🔗 Repository
[https://github.com/680pxl/blender_pbr_importer](https://github.com/680pxl/blender_pbr_importer)

---

## ✨ Features

### 🚀 Automation

* **Direct ZIP Import**
  Select a `.zip` file — the addon extracts it, locates textures and models, and sets everything up instantly.

* **Auto-Mesh Detection**
  Automatically imports 3D files (`.fbx`, `.obj`, `.gltf`, `.glb`) found inside folders or ZIP archives.

* **LOD Support**
  Prioritizes **LOD0** when multiple Levels of Detail are detected.

* **Decal / Plane Mode**
  If no mesh is found and nothing is selected, a correctly scaled plane is created based on texture dimensions
  (perfect for decals, walls, or surfaces).

---

### 🧠 Smart Shader Setup

Uses **strict texture name matching** to prevent common mistakes
(e.g. confusing Normal maps with Roughness maps).

#### Supported Channels

* Albedo / Diffuse / Base Color
* Ambient Occlusion (AO) — automatically multiplied with Albedo
* Roughness / Metallic
* Normal Map (OpenGL / DirectX) / Bump
* Transmission / Opacity / Alpha
* Displacement / Height

#### Auto-Fixes

* Sets **Color Space to Non-Color** for data textures
  (Roughness, Normal, Metallic)
* Enables **Hashed Alpha / Shadow Mode** for transparent textures
  (Cycles / Eevee)
* Adds **Mapping Nodes** for global texture scaling

---

## 🔧 Editions

### 1️⃣ Cycles / Eevee Importer

* **Shader:** Principled BSDF
* **Compatibility:** Optimized for Blender 4.2+
  (works with 3.x, uses new 4.0+ sockets like *Transmission Weight* where available)
* **Displacement:** Automatically enables *Displacement Only*

---

### 2️⃣ Octane Render Importer

* **Shader:** Universal Material
* **Nodes:** Automatically locates correct Octane nodes
  (Octane Image Tex, Octane Transform, etc.)
* **Requirement:** Blender Octane Edition Plugin

---

## 📥 Installation

1. Download the required Python file:

   * `cycles_pbr_loader.py` — Cycles / Eevee
   * `octane_pbr_importer.py` — Octane Render

2. In Blender:

   * Go to **Edit / Preferences / Add-ons**
   * Click **Install...**
   * Select the downloaded file
   * Enable the addon checkbox

---

## 🎮 Usage

The addon integrates into **File / Import** and supports hotkeys.

### Method A: Menu

Go to **File / Import**

* Cycles PBR (Auto ZIP)
* Cycles PBR (Auto Folder)
* Octane PBR (Auto ZIP)
* Octane PBR (Auto Folder)

### Method B: Hotkey (Quick Import)

* **Default Hotkey:** `Shift + W` (3D View)
* Opens the file browser directly in ZIP import mode

---

## 📝 Known Issues

* Texture detection may fail when channel keywords are part of long filenames
  (e.g. `corrugated_metal_sheet`).
  ➜ Use clearer naming conventions.
* **Cycles only:** Incorrect node setup for transmission textures.

---

## 📸 Links

* Instagram: [https://instagram.com/680pxl](https://instagram.com/680pxl)

bauen 😌
