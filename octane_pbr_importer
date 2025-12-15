bl_info = {
    "name": "Octane PBR Importer",
    "author": "680pxl",
    "version": (2, 0),
    "blender": (4, 5, 2),
    "location": "File > Import > Octane PBR...",
    "description": "Instantly turns PBR ZIPs into perfect Octane materials",
    "warning": "Requires Octane Render",
    "category": "Import-Export",
}

import bpy
import os
import zipfile
from bpy_extras.io_utils import ImportHelper
from bpy.types import Operator, AddonPreferences

# ==============================================================================
# --- 1. CORE LOGIC (SHARED) ---
# ==============================================================================

def find_octane_node_id(include_terms, exclude_terms=None):
    if exclude_terms is None: exclude_terms = []
    candidates = []
    for cls in bpy.types.Node.__subclasses__():
        if hasattr(cls, 'bl_idname'):
            idname = cls.bl_idname
            if all(term.lower() in idname.lower() for term in include_terms):
                if not any(ex.lower() in idname.lower() for ex in exclude_terms):
                    candidates.append(idname)
    candidates.sort(key=lambda x: (not "ShaderNode" in x, len(x)))
    return candidates[0] if candidates else None

def create_tex_node(nodes, img_id, filepath, x_pos, y_pos, transform_node, links, is_float=False):
    try:
        node = nodes.new(type=img_id)
        img = bpy.data.images.load(filepath)
        if is_float and hasattr(img, 'colorspace_settings'):
            img.colorspace_settings.name = 'Non-Color'
        if hasattr(node, 'image'): node.image = img
        node.location = (x_pos, y_pos)
        
        if transform_node:
            for inp in node.inputs:
                if "trans" in inp.name.lower() or "proj" in inp.name.lower():
                    links.new(transform_node.outputs[0], inp)
                    break
        return node
    except: return None

def link_socket(univ_node, links, tex_node, target_names, exclude_names=None):
    if not tex_node: return False
    if exclude_names is None: exclude_names = []
    for inp in univ_node.inputs:
        if hasattr(inp, 'bl_idname') and "grouptile" in inp.bl_idname.lower(): continue
        if any(t.lower() in inp.name.lower() for t in target_names):
            if not any(ex.lower() in inp.name.lower() for ex in exclude_names):
                links.new(tex_node.outputs[0], inp)
                return True
    return False

# --- STRICT MATCHING HELPER ---
def matches_channel(filename, valid_terms, banned_terms):
    f = filename.lower()
    # 1. Must contain at least one valid term
    if not any(t in f for t in valid_terms):
        return False
    # 2. Must NOT contain any banned term
    if any(b in f for b in banned_terms):
        return False
    return True

def create_and_assign_material(target_objects, folder_path):
    if not target_objects: return {"WARNING": "No target objects found."}

    mat_name = os.path.basename(os.path.normpath(folder_path))
    if not mat_name: mat_name = "Octane_Imported_Mat"

    mat = bpy.data.materials.new(name=mat_name)
    mat.use_nodes = True
    tree = mat.node_tree
    nodes = tree.nodes
    links = tree.links
    nodes.clear()

    # --- Node IDs ---
    univ_id = find_octane_node_id(['Universal', 'Material'], ['Camera', 'Imager', 'Output'])
    img_id = find_octane_node_id(['Image', 'Oct'], ['Imager', 'Output', 'Environment', 'Viewer'])
    if not img_id: img_id = find_octane_node_id(['Image'], ['Imager', 'Output', 'Environment'])
    trans_id = find_octane_node_id(['Transform', '3D'], ['Output', 'Camera'])
    disp_node_id = find_octane_node_id(['Displacement', 'Texture'], ['Mix', 'Vertex'])
    mult_id = find_octane_node_id(['Multiply', 'Texture'], ['Math', 'Value', 'Matrix'])
    
    if not univ_id or not img_id: return {"ERROR": "Octane Nodes not found."}

    # --- Basic Layout ---
    try:
        output_node = nodes.new(type='ShaderNodeOutputMaterial')
        output_node.location = (600, 0)
        
        univ_node = nodes.new(type=univ_id)
        univ_node.location = (200, 0)
        
        links.new(univ_node.outputs[0], output_node.inputs['Surface'])
        
        transform_node = None
        if trans_id:
            transform_node = nodes.new(type=trans_id)
            transform_node.label = "MASTER SCALE"
            transform_node.location = (-1200, 200)
    except: return {"ERROR": "Error during node setup."}

    found_files = []
    valid_img_ext = ('.jpg', '.png', '.exr', '.tif', '.tiff', '.jpeg')
    for root, _, files in os.walk(folder_path):
        for f in files:
            if f.lower().endswith(valid_img_ext):
                found_files.append((f, os.path.join(root, f)))

    processed = []
    albedo_node = None
    ao_node = None

    current_y = 400
    y_step = 280 
    tex_x = -600

    # --- STRICT DEFINITIONS ---
    GLOBAL_BANS = ["billboard", "preview", "thumb"]
    
    # Specific Bans to prevent cross-loading
    BANNED_METAL = ["cavity", "curvature", "ao", "ambient", "diffuse", "albedo", "color", "rough", "spec", "norm", "bump", "height", "disp", "mask"]
    BANNED_ROUGH = ["cavity", "curvature", "metal", "spec", "norm", "bump", "diffuse", "albedo"]
    BANNED_ALBEDO = ["ao", "ambient", "cavity", "curvature", "rough", "metal", "spec", "norm", "bump", "height", "disp", "mask"]
    BANNED_NORMAL = ["cavity", "curvature", "bump", "spec", "rough", "metal", "ao"]
    BANNED_SPEC = ["cavity", "curvature", "rough", "metal", "ao", "norm", "bump"]

    for fname, fpath in found_files:
        f = fname.lower()
        if any(b in f for b in GLOBAL_BANS): continue

        # 1. ALBEDO
        if matches_channel(f, ["albedo", "diffuse", "color"], BANNED_ALBEDO) and "Albedo" not in processed:
            albedo_node = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, False)
            processed.append("Albedo")
            current_y -= y_step

        # 2. AO
        elif matches_channel(f, ["ao", "ambient", "occlusion"], ["rough", "metal", "spec", "norm", "bump"]) and "AO" not in processed:
            ao_node = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            processed.append("AO")
            current_y -= y_step

        # 3. SPECULAR (Index 10)
        elif matches_channel(f, ["specular", "spec"], BANNED_SPEC) and "Specular" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            try:
                if len(univ_node.inputs) > 10:
                    links.new(n.outputs[0], univ_node.inputs[10])
                    processed.append("Specular")
            except: pass

        # 4. ROUGHNESS (Index 13)
        elif matches_channel(f, ["roughness", "rough"], BANNED_ROUGH) and "Roughness" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            try:
                if len(univ_node.inputs) > 13:
                    links.new(n.outputs[0], univ_node.inputs[13])
                    processed.append("Roughness")
            except: pass
        
        # 5. METALLIC (Index 7) - STRICT
        elif matches_channel(f, ["metallic", "metalness", "_metal", "-metal"], BANNED_METAL) and "Metal" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            try:
                if len(univ_node.inputs) > 7:
                    links.new(n.outputs[0], univ_node.inputs[7])
                    processed.append("Metal")
            except: pass

        # 6. NORMAL
        elif matches_channel(f, ["normal"], BANNED_NORMAL) and "Normal" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            if link_socket(univ_node, links, n, ["Normal"], ["Coating", "Sheen", "Bump"]): 
                processed.append("Normal")
        
        # 7. BUMP (Index 48)
        elif matches_channel(f, ["bump"], ["normal", "cavity", "curvature"]) and "Bump" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            try:
                if len(univ_node.inputs) > 48:
                    links.new(n.outputs[0], univ_node.inputs[48])
                    processed.append("Bump")
            except: pass

        # 8. TRANSMISSION (Index 1)
        elif matches_channel(f, ["trans"], ["opacity", "rough"]) and "Transmission" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            try:
                if len(univ_node.inputs) > 1:
                    links.new(n.outputs[0], univ_node.inputs[1])
                    processed.append("Transmission")
            except: pass

        # 9. OPACITY
        elif matches_channel(f, ["opacity", "alpha"], ["trans"]) and "Opacity" not in processed:
            n = create_tex_node(nodes, img_id, fpath, tex_x, current_y, transform_node, links, True)
            current_y -= y_step
            link_socket(univ_node, links, n, ["Opacity", "Alpha"], ["Transmission"])
            processed.append("Opacity")

        # 10. DISPLACEMENT
        elif matches_channel(f, ["height", "disp"], ["normal", "bump"]) and "Disp" not in processed:
            if disp_node_id:
                tex_node = create_tex_node(nodes, img_id, fpath, tex_x - 300, current_y, transform_node, links, True)
                disp_node = nodes.new(type=disp_node_id)
                disp_node.location = (tex_x, current_y)
                disp_node.label = "DISP CONTROL"
                
                # Set Default Height 0.05
                for inp in disp_node.inputs:
                    if "amount" in inp.name.lower() or "height" in inp.name.lower():
                        inp.default_value = 0.05
                
                current_y -= y_step
                
                for inp in disp_node.inputs:
                    if hasattr(inp, 'bl_idname') and "grouptile" in inp.bl_idname.lower(): continue
                    if "texture" in inp.name.lower():
                        links.new(tex_node.outputs[0], inp); break
                for inp in univ_node.inputs:
                    if hasattr(inp, 'bl_idname') and "grouptile" in inp.bl_idname.lower(): continue
                    if "displacement" in inp.name.lower():
                        links.new(disp_node.outputs[0], inp)
                        processed.append("Disp"); break

    # --- ALBEDO * AO MIX ---
    if albedo_node:
        if ao_node and mult_id:
            mult_node = nodes.new(type=mult_id)
            mult_node.location = (-300, albedo_node.location.y)
            mult_node.label = "Albedo * AO"
            if len(mult_node.inputs) >= 2:
                links.new(albedo_node.outputs[0], mult_node.inputs[0])
                links.new(ao_node.outputs[0], mult_node.inputs[1])
                link_socket(univ_node, links, mult_node, ["Albedo", "Diffuse", "Base Color"], ["Coating", "Sheen", "Transmission", "Specular"])
            else:
                link_socket(univ_node, links, albedo_node, ["Albedo", "Diffuse", "Base Color"], ["Coating", "Sheen", "Transmission", "Specular"])
        else:
            link_socket(univ_node, links, albedo_node, ["Albedo", "Diffuse", "Base Color"], ["Coating", "Sheen", "Transmission", "Specular"])

    # Assign
    for obj in target_objects:
        if obj.type == 'MESH':
            if not obj.data.materials: obj.data.materials.append(mat)
            else: obj.data.materials[0] = mat

    return {"SUCCESS": "Import successful."}

def import_mesh_files_multilod(folder_path):
    extensions = ('.fbx', '.obj', '.gltf', '.glb')
    all_meshes = []
    for root, _, files in os.walk(folder_path):
        for f in files:
            if f.lower().endswith(extensions):
                all_meshes.append(os.path.join(root, f))
    
    if not all_meshes: return False, []

    has_lod = any("lod" in os.path.basename(p).lower() for p in all_meshes)
    final_list = [p for p in all_meshes if "lod0" in os.path.basename(p).lower()] if has_lod else all_meshes
    if has_lod and not final_list and all_meshes: final_list = [all_meshes[0]]
    if not final_list: return False, []

    imported_objects = []
    for filepath in final_list:
        objs_before = set(bpy.context.scene.objects)
        try:
            l = filepath.lower()
            if l.endswith('.fbx'): bpy.ops.import_scene.fbx(filepath=filepath)
            elif l.endswith('.obj'): 
                if hasattr(bpy.ops.wm, 'obj_import'): bpy.ops.wm.obj_import(filepath=filepath)
                else: bpy.ops.import_scene.obj(filepath=filepath)
            elif l.endswith(('.gltf', '.glb')): bpy.ops.import_scene.gltf(filepath=filepath)
        except: continue
        imported_objects.extend(list(set(bpy.context.scene.objects) - objs_before))

    mesh_objects = [o for o in imported_objects if o.type == 'MESH']
    
    if mesh_objects:
        bpy.ops.object.select_all(action='DESELECT')
        for o in mesh_objects: o.select_set(True)
        bpy.context.view_layer.objects.active = mesh_objects[0]

    return bool(mesh_objects), mesh_objects

def create_image_plane_from_folder(folder_path):
    valid_ext = ('.jpg', '.png', '.exr', '.tif', '.tiff')
    target_img_path = None
    
    for root, _, files in os.walk(folder_path):
        for f in files:
            if f.lower().endswith(valid_ext) and "billboard" not in f.lower():
                if any(x in f.lower() for x in ["albedo", "diffuse", "color"]):
                    target_img_path = os.path.join(root, f)
                    break
        if target_img_path: break
    
    if not target_img_path:
        for root, _, files in os.walk(folder_path):
            for f in files:
                if f.lower().endswith(valid_ext):
                    target_img_path = os.path.join(root, f); break
            if target_img_path: break

    if not target_img_path:
        bpy.ops.mesh.primitive_plane_add(size=2)
        return bpy.context.active_object

    img = bpy.data.images.load(target_img_path)
    if img.size[1] == 0: return None
    aspect = img.size[0] / img.size[1]
    
    bpy.ops.mesh.primitive_plane_add(size=2)
    plane = bpy.context.active_object
    plane.name = "Decal_" + os.path.basename(folder_path)
    plane.scale.x = aspect
    plane.scale.y = 1.0
    return plane

# --- MAIN LOGIC WRAPPER ---
def run_import_logic(self, context, directory):
    # 1. Mesh
    has_mesh, new_objects = import_mesh_files_multilod(directory)
    target_objects = []

    if has_mesh:
        target_objects = new_objects
    else:
        # 2. Selection
        selected = context.selected_objects
        if selected:
            target_objects = selected
        else:
            # 3. Decal
            plane = create_image_plane_from_folder(directory)
            if plane: target_objects = [plane]

    if not target_objects:
        self.report({'ERROR'}, "Nothing imported and nothing selected.")
        return {'CANCELLED'}

    # 4. Material
    result = create_and_assign_material(target_objects, directory)
    
    if "ERROR" in result:
        self.report({'ERROR'}, result["ERROR"])
        return {'CANCELLED'}
    
    self.report({'INFO'}, result["SUCCESS"])
    return {'FINISHED'}


# ==============================================================================
# --- 2. OPERATORS ---
# ==============================================================================

class IMPORT_OT_octane_pbr_zip(Operator, ImportHelper):
    """Import ZIP File (Auto Mesh/Decal)"""
    bl_idname = "import_scene.octane_pbr_zip"
    bl_label = "Octane PBR (Auto ZIP)"
    bl_options = {'REGISTER', 'UNDO'}
    filter_glob: bpy.props.StringProperty(default="*.zip", options={'HIDDEN'})

    def execute(self, context):
        zip_path = self.filepath
        extract_dir = os.path.splitext(zip_path)[0]
        if not os.path.exists(extract_dir):
            try:
                with zipfile.ZipFile(zip_path, 'r') as zip_ref: zip_ref.extractall(extract_dir)
            except: 
                self.report({'ERROR'}, "Error extracting ZIP.")
                return {'CANCELLED'}
        return run_import_logic(self, context, extract_dir)

class IMPORT_OT_octane_pbr_folder(Operator):
    """Import Folder (Auto Mesh/Decal)"""
    bl_idname = "import_scene.octane_pbr_folder"
    bl_label = "Octane PBR (Auto Folder)"
    bl_options = {'REGISTER', 'UNDO'}
    directory: bpy.props.StringProperty(name="Directory", options={'HIDDEN'}, subtype='DIR_PATH')
    filter_folder: bpy.props.BoolProperty(default=True, options={'HIDDEN'})

    def invoke(self, context, event):
        context.window_manager.fileselect_add(self)
        return {'RUNNING_MODAL'}

    def execute(self, context):
        return run_import_logic(self, context, self.directory)


# ==============================================================================
# --- 3. UI & REGISTRATION ---
# ==============================================================================

class OctaneImportPreferences(AddonPreferences):
    bl_idname = __name__

    def draw(self, context):
        layout = self.layout
        layout.label(text="Hotkey for ZIP Import (Default: Shift+W):")
        wm = context.window_manager
        kc = wm.keyconfigs.user
        found = False
        for km in kc.keymaps:
            if km.name == '3D View':
                for kmi in km.keymap_items:
                    if kmi.idname == IMPORT_OT_octane_pbr_zip.bl_idname:
                        layout.context_pointer_set("keymap", km)
                        layout.prop(kmi, "type", text="Hotkey", full_event=True)
                        found = True; break
        if not found: layout.label(text="Hotkey check failed.")

addon_keymaps = []

def menu_func_zip(self, context):
    self.layout.operator(IMPORT_OT_octane_pbr_zip.bl_idname, text="Octane PBR (Auto ZIP)")

def menu_func_folder(self, context):
    self.layout.operator(IMPORT_OT_octane_pbr_folder.bl_idname, text="Octane PBR (Auto Folder)")

def register():
    bpy.utils.register_class(OctaneImportPreferences)
    bpy.utils.register_class(IMPORT_OT_octane_pbr_zip)
    bpy.utils.register_class(IMPORT_OT_octane_pbr_folder)
    bpy.types.TOPBAR_MT_file_import.append(menu_func_zip)
    bpy.types.TOPBAR_MT_file_import.append(menu_func_folder)
    
    wm = bpy.context.window_manager
    kc = wm.keyconfigs.addon
    if kc:
        km = kc.keymaps.new(name='3D View', space_type='VIEW_3D')
        kmi = km.keymap_items.new(IMPORT_OT_octane_pbr_zip.bl_idname, 'W', 'PRESS', shift=True)
        addon_keymaps.append((km, kmi))

def unregister():
    for km, kmi in addon_keymaps: km.keymap_items.remove(kmi)
    addon_keymaps.clear()
    bpy.types.TOPBAR_MT_file_import.remove(menu_func_zip)
    bpy.types.TOPBAR_MT_file_import.remove(menu_func_folder)
    bpy.utils.unregister_class(IMPORT_OT_octane_pbr_folder)
    bpy.utils.unregister_class(IMPORT_OT_octane_pbr_zip)
    bpy.utils.unregister_class(OctaneImportPreferences)

if __name__ == "__main__":
    register()
