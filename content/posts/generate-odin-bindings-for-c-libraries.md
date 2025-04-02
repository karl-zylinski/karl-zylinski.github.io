---
title: "Generate Odin bindings for C libraries"
date: 2025-04-02T19:30:00+01:00

cover:
  image: "bindgen/cover.png"
---

I've written a binding generator for Odin. You can check it out here: https://github.com/karl-zylinski/odin-c-bindgen

In this blog post we'll look at what it is and how to use it.

## What is it?

This binding generator takes header files from C libraries and outputs Odin code. This Odin code can then be used to interface with the library. This code is known as "binding": It glues the world of Odin together with that of the C library.

I've put effort into making sure that the generated bindings look good and are easy on the eye. I also try to make it possible to use Odin-specific features, such as `bit_set`.

## Why would one need it?

Some people argue that it is best to make bindings by hand. It's true that you often can create better bindings, more tailored to Odin that way. But there are some cases where a binding generator is good:
- You want to quickly test a library in your Odin code base.
- You want to compare several libraries. Making bindings by hand might make you settle for a less good choice because you only had energy to make bindings for one of the libraries.
- You want to write bindings by hand, but you want to get a good "basis" to start from.

## Example: Generated raylib bindings

I know Odin comes with Raylib bindings, but I generated some just to test the generator. It's nice to have the high-quality official bindings to compare against. You can find the generated bindings here: https://github.com/karl-zylinski/odin-c-bindgen/tree/main/examples/raylib.

In order to generate the bindings you would:
- Install `clang` (https://llvm.org/, used for analysing the C headers)
- Download the binding generator repository
- Build the binding generator: `odin build src -out:bindgen.exe`
- Run the binding generator and feed it the `raylib` folder: `bindgen.exe examples/raylib`
- (optional) Test the bindings using the program in `examples/raylib/test`.

Below I'll talk about how to configure the generator.

### The `bindgen.sjson` configuration file

The binding generator will look for a file called `bindgen.sjson` inside the folder you specify.

This file configures the binding generator. In it you can tell the generator to strip some prefixes from type names, override some procedure parameter types, "bitsetify" enums, etc!

The `bindgen.sjson` file for the raylib example looks like follows:

```c
// Inputs can be folders or files. It will look for header (.h) files inside
// any folder. The bindings will be based on those headers. Also, any .lib,
// .odin, .dll etc will be copied to the output folder.
inputs = [
	"input"
]

// Output folder: One .odin file per processed header
output_folder = "raylib"

// Remove this prefix from types names (structs, enums, etc)
remove_type_prefix = ""

// Remove this prefix from function names (and add it as link_prefix) to the foreign group
remove_function_prefix = ""

// Only include things that has this prefix
required_prefix = ""

// Single lib file to import
import_lib = "" // For example: "some_lib.lib"

// Code file that contain libray import code and whatever else extra you need.
// Overrides lib_file. Is pasted near top of the final bindings.
imports_file = "imports.odin"

// For package line at top of output files
package_name = "raylib"

// "Old_Name" = "New_Name",
rename_types = {
	"ConfigFlags" = "ConfigFlag"
}

// Turns an enum into a bit_set. Converts the values of the enum into
// appropriate values for a bit_set. Creates a bit_set type that uses the enum.
// Properly removes enum values with value 0. Translates the enum values using
// a log2 procedure.
bit_setify = {
	// "Pre_Existing_Enum_Type" = "New_Bit_Set_Type"
	"Gesture" = "Gestures"
	"ConfigFlag" = "ConfigFlags"
}

// Completely override the definition of a type.
type_overrides = {
	"Vector2" = "[2]f32"
	"Vector3" = "[3]f32"
	"Vector4" = "[4]f32"
	"Matrix"  = "#row_major matrix[4, 4]f32"
	"Color"   = "distinct [4]u8"
}

// Override the type of a struct field. Note that a plain `[^]` can be used to
// modify the existing type.
struct_field_overrides = {
	"Image.format"      = "PixelFormat"
	"Texture.format"    = "PixelFormat"
	"NPatchInfo.layout" = "NPatchLayout"
	"GlyphInfo.value"   = "rune"

	"Mesh.vertices"     = "[^]"
	"Mesh.texcoords"    = "[^]"
	"Mesh.texcoords2"   = "[^]"
	"Mesh.normals"      = "[^]"
	"Mesh.tangents"     = "[^]"
	"Mesh.colors"       = "[^]"
	"Mesh.indices"      = "[^]"
	"Mesh.animVertices" = "[^]"
	"Mesh.animNormals"  = "[^]"
	"Mesh.boneIds"      = "[^]"
	"Mesh.boneWeights"  = "[^]"
	"Mesh.boneMatrices" = "[^]"
	"Mesh.vboId"        = "[^]"
	"Shader.locs"       = "[^]"
	"Material.maps"     = "[^]"
	"Model.meshes"        = "[^]"
	"Model.materials"     = "[^]"
	"Model.meshMaterials" = "[^]"
	"Model.bones"         = "[^]"
	"Model.bindPose"      = "[^]"
	"ModelAnimation.bones"      = "[^]"
	"ModelAnimation.framePoses" = "[^][^]Transform"

	// This is not a complete override list, it's just an example.
}

// Overrides the type of a procedure parameter or return value. For a parameter
// use the key Proc_Name.parameter_name. For a return value use the key Proc_Name.
// Note that a plain `[^]` and `#by_ptr` can be used to modify the existing type.
procedure_type_overrides = {
	"SetConfigFlags.flags"         = "ConfigFlags"
	"IsKeyPressed.key"             = "KeyboardKey"
	"IsKeyPressedRepeat.key"       = "KeyboardKey"
	"IsKeyDown.key"                = "KeyboardKey"
	"IsKeyReleased.key"            = "KeyboardKey"
	"IsKeyUp.key"                  = "KeyboardKey"
	"GetKeyPressed"                = "KeyboardKey"
	"IsMouseButtonPressed.button"  = "MouseButton"
	"IsMouseButtonDown.button"     = "MouseButton"
	"IsMouseButtonReleased.button" = "MouseButton"
	"IsMouseButtonUp.button"       = "MouseButton"
	"SetShaderValue.uniformType"   = "ShaderUniformDataType"
	"SetShaderValueV.uniformType"  = "ShaderUniformDataType"
	"SetExitKey.key"               = "KeyboardKey"
	"GetKeyName.key"               = "KeyboardKey"
	"IsGestureDetected.gesture"    = "Gestures"
	"SetGesturesEnabled.flags"     = "Gestures"
}

// Inject a new type before another type. Use `rename_types` to just rename
// a pre-existing type.
inject_before = {
	// "Some_Type" = "New_Type :: distinct int"
}

// For typedefs that don't resolve to anything: Put them in here to create
// empty structs with that name.
opaque_types = [
	// "Some_Type"
]

// additional include paths to send into clang. While generating the bindings
// clang will look into this path in search for included headers.
clang_include_paths = [
	// "some_path"
]

// Writes the clang JSON ast dump for debug inspection
debug_dump_json_ast = false
```

Let's look a bit closer at a few of the things in there:

---

```c
inputs = [
	"input"
]
```
This will make the generator search the `input` folder for C header files. Any header file in there will be sent into `clang`. Clang will generate an abstract syntax tree from it, which we use to output the Odin binding code.

It'll also copy any `.odin`, `.lib`, `.a`, `.dll` or `.dylib` in the input folder. So you can put thing you want to "bring along" into your bindings in there.

> Sometimes bindings need some additional, hand-written code. You can put that in an Odin file inside the input folder.

---

```
import_lib = ""

imports_file = "imports.odin"
```
You can write `import_lib = "raylib.lib"` in order to add a `foreign import lib { "raylib.lib" }` line to the bindings. But if you want some additional set up, then you can also write your own little "imports" file. That's what the `imports_file` thing does: It will paste in the contents of that file near the top of your bindings. In there you can do custom platform-specific logic for library file imports, etc.


---

```c
rename_types = {
	"ConfigFlags" = "ConfigFlag"
}

bit_setify = {
	"Gesture" = "Gestures"
	"ConfigFlag" = "ConfigFlags"
}
```

The first part renames the type `ConfigFlags` to `ConfigFlag`. That's done because we want to use that type for a `bit_set`. The second part says to create a `bit_set` called `ConfigFlags` that wraps the `ConfigFlag` enum. It also bit setifies the `Gesture` enum under the name `Gestures`.

The bitsetification of `Gesture` will take the following C header code and turn it into the Odin code;

C code:
```c
typedef enum {
    GESTURE_NONE        = 0,        // No gesture
    GESTURE_TAP         = 1,        // Tap gesture
    GESTURE_DOUBLETAP   = 2,        // Double tap gesture
    GESTURE_HOLD        = 4,        // Hold gesture
    GESTURE_DRAG        = 8,        // Drag gesture
    GESTURE_SWIPE_RIGHT = 16,       // Swipe right gesture
    GESTURE_SWIPE_LEFT  = 32,       // Swipe left gesture
    GESTURE_SWIPE_UP    = 64,       // Swipe up gesture
    GESTURE_SWIPE_DOWN  = 128,      // Swipe down gesture
    GESTURE_PINCH_IN    = 256,      // Pinch in gesture
    GESTURE_PINCH_OUT   = 512       // Pinch out gesture
} Gesture;
```

Odin Code:
```c
// Gesture
// NOTE: Provided as bit-wise flags to enable only desired gestures
Gesture :: enum c.int {
	TAP         = 0, // Tap gesture
	DOUBLETAP   = 1, // Double tap gesture
	HOLD        = 2, // Hold gesture
	DRAG        = 3, // Drag gesture
	SWIPE_RIGHT = 4, // Swipe right gesture
	SWIPE_LEFT  = 5, // Swipe left gesture
	SWIPE_UP    = 6, // Swipe up gesture
	SWIPE_DOWN  = 7, // Swipe down gesture
	PINCH_IN    = 8, // Pinch in gesture
	PINCH_OUT   = 9, // Pinch out gesture
}

Gestures :: distinct bit_set[Gesture; c.int]
```

Note how it has added a `Gestures` type and also modified the values of each enum member. The values are modified so that the `bit_set` works correctly.

---

```c
type_overrides = {
	"Vector2" = "[2]f32"
	"Vector3" = "[3]f32"
	"Vector4" = "[4]f32"
	"Matrix"  = "#row_major matrix[4, 4]f32"
	"Color"   = "distinct [4]u8"
}
```
Some types in are better replaced completely. In raylib the vectors and color types are structs. It's nice to instead use fixed arrays, so that you can do array programming swizzling (`some_vector.xyz` and `some_color.rgb` etc).

---

```c
struct_field_overrides = {
	"Image.format"      = "PixelFormat"
	"Texture.format"    = "PixelFormat"
	"NPatchInfo.layout" = "NPatchLayout"
	"GlyphInfo.value"   = "rune"

	"Mesh.vertices"     = "[^]"
	"Mesh.texcoords"    = "[^]"
	"Mesh.texcoords2"   = "[^]"
	"Mesh.normals"      = "[^]"
	"Mesh.tangents"     = "[^]"
	"Mesh.colors"       = "[^]"
	"Mesh.indices"      = "[^]"
	"Mesh.animVertices" = "[^]"
	"Mesh.animNormals"  = "[^]"
	"Mesh.boneIds"      = "[^]"
	"Mesh.boneWeights"  = "[^]"
	"Mesh.boneMatrices" = "[^]"
	"Mesh.vboId"        = "[^]"
	"Shader.locs"       = "[^]"
	"Material.maps"     = "[^]"
	"Model.meshes"        = "[^]"
	"Model.materials"     = "[^]"
	"Model.meshMaterials" = "[^]"
	"Model.bones"         = "[^]"
	"Model.bindPose"      = "[^]"
	"ModelAnimation.bones"      = "[^]"
	"ModelAnimation.framePoses" = "[^][^]Transform"

	// This is not a complete override list, it's just an example.
}
```

This one is important: As you can see you can override the type of a struct field. `"Image.format" = "PixelFormat"` will make the `format` field of the `Image` struct use the type `PixelFormat`. Raylib uses an integer for the type, but in Odin it makes sense to use the actual enum type.

Another important thing are the lines that looks like so:
```c
"Mesh.vertices" = "[^]"
```

This `Mesh` type looks like this in C:
```c
typedef struct Mesh {
	int vertexCount;        // Number of vertices stored in arrays
	int triangleCount;      // Number of triangles stored (indexed or not)

	// Vertex attributes data
	float *vertices;        // Vertex position (XYZ - 3 components per vertex) (shader-location = 0)
	float *texcoords;       // Vertex texture coordinates (UV - 2 components per vertex) (shader-location = 1)
	float *texcoords2;      // Vertex texture second coordinates (UV - 2 components per vertex) (shader-location = 5)
	float *normals;         // Vertex normals (XYZ - 3 components per vertex) (shader-location = 2)
	float *tangents;        // Vertex tangents (XYZW - 4 components per vertex) (shader-location = 4)
	unsigned char *colors;      // Vertex colors (RGBA - 4 components per vertex) (shader-location = 3)
	unsigned short *indices;    // Vertex indices (in case vertex data comes indexed)

	// Animation vertex data
	float *animVertices;    // Animated vertex positions (after bones transformations)
	float *animNormals;     // Animated normals (after bones transformations)
	unsigned char *boneIds; // Vertex bone ids, max 255 bone ids, up to 4 bones influence by vertex (skinning) (shader-location = 6)
	float *boneWeights;     // Vertex bone weight, up to 4 bones influence by vertex (skinning) (shader-location = 7)
	Matrix *boneMatrices;   // Bones animated transformation matrices
	int boneCount;          // Number of bones

	// OpenGL identifiers
	unsigned int vaoId;     // OpenGL Vertex Array Object id
	unsigned int *vboId;    // OpenGL Vertex Buffer Objects id (default vertex data)
} Mesh;
```

The bindings for fields such as `vertices` would by default use the type `^f32`. But that field is actually supposed to be a C-style array. Such arrays are best represented as multi-pointers. So writing
```c
"Mesh.vertices" = "[^]"
```
Will swap out that `^` in `^f32` for `[^]f32`, so the field in the generated Odin code for `Mesh` will contain a vertices field that looks like so:
```
vertices: [^]f32
```

For complete Raylib bindings the list of struct field type overrides would be longer than in this example. If you're making bindings for a library, then I recommend: Just generate them and add overrides like this as you notice you need them. You can make the binding generator run as part of your build process, before building your program that uses the bindings, so you can quickly add these tweaks.

---

```c
procedure_type_overrides = {
	"SetConfigFlags.flags"         = "ConfigFlags"
	"IsKeyPressed.key"             = "KeyboardKey"
	"IsKeyPressedRepeat.key"       = "KeyboardKey"
	"IsKeyDown.key"                = "KeyboardKey"
	"IsKeyReleased.key"            = "KeyboardKey"
	"IsKeyUp.key"                  = "KeyboardKey"
	"GetKeyPressed"                = "KeyboardKey"
	"IsMouseButtonPressed.button"  = "MouseButton"
	"IsMouseButtonDown.button"     = "MouseButton"
	"IsMouseButtonReleased.button" = "MouseButton"
	"IsMouseButtonUp.button"       = "MouseButton"
	"SetShaderValue.uniformType"   = "ShaderUniformDataType"
	"SetShaderValueV.uniformType"  = "ShaderUniformDataType"
	"SetExitKey.key"               = "KeyboardKey"
	"GetKeyName.key"               = "KeyboardKey"
	"IsGestureDetected.gesture"    = "Gestures"
	"SetGesturesEnabled.flags"     = "Gestures"
}
```

Similarly to the struct type overrides, you can also override the type of procedure parameters and return values. Use the key `Procedure_Name.parameter` to override a parameter. Just write the procedure name to override the return value.

On the right side, you can also write `[^]` and `#by_ptr` instead of a complete type name. That'll modify the current type, just like with struct field type overrides.


## Another example: Generated `ufbx` bindings

For another example, you can have a look at this generator setup for the `ufbx` library: https://github.com/karl-zylinski/odin-c-bindgen/tree/main/examples/ufbx.

![A program written using raylib that displays an FBX model. The FBX model has been loaded using the generated `ufbx` bindings](/bindgen/ufbx.png)

The picture shows a test program that is included with this example. It uses the generated `ufbx` bindings to load a model and display it using raylib.

> FBX is a common format for 3D models.

The `bindgen.sjson` used here is a bit different from the one we used for raylib:
```
inputs = [
    "."
]

remove_type_prefix = "ufbx_"

remove_function_prefix = "ufbx_"

import_lib = "ufbx.lib"

package_name = "ufbx"

output_folder = "ufbx"

force_ada_case_types = true

type_overrides = {
    "Vec2" = "[2]Real"
    "Vec3" = "[3]Real"
    "Vec4" = "[4]Real"

    // Assumes UFBX_REAL_IS_FLOAT to be set below, so that `Real :: f32`
    "Quat" = "quaternion128"
}

clang_defines = {
    "UFBX_REAL_IS_FLOAT" = "1"
}

struct_field_overrides = {
    "Node_List.data" = "[^]"
    "Face_List.data" = "[^]"
    "Uint32_List.data" = "[^]"
    "Vec2_List.data" = "[^]"
    "Vec3_List.data" = "[^]"
    "Vec4_List.data" = "[^]"
}

procedure_type_overrides = {
    "generate_indices.streams" = "[^]"
}

rename_types = {
    "Prop_Flags" = "Prop_Flag"
    "Transform_Flags" = "Transform_Flag"
    "Baked_Key_Flags" = "Baked_Key_Flag"
}

bit_setify = {
    "Prop_Flag" = "Prop_Flags"
    "Transform_Flag" = "Transform_Flags"
    "Baked_Key_Flag" = "Baked_Key_Flags"
}

```

---

```c
remove_type_prefix = "ufbx_"

remove_function_prefix = "ufbx_"
```

This will strip the `ufbx_` prefixes that are everywhere in `ufbx`. Since the library will be loaded as a package, that package will have the `ufbx` namespace, so removing it makes everything more tidy.

---

```
clang_defines = {
    "UFBX_REAL_IS_FLOAT" = "1"
}
```
This sends along some defines to clang, so that it applies them when generating the bindings.

---

```c
force_ada_case_types = true
```

This will make all types use ada case. I.e. `Look_Like_This`. In most cases I recommend staying with the library's own convention. But `ufbx` had many names such as `ufbx_string`. When the `ufbx_` prefix was removed it caused a conflict with Odin's own `string` type name. So in this case I found it best to force ada case.

## More examples and additional help

See the examples folder for additional examples: https://github.com/karl-zylinski/odin-c-bindgen/tree/main/examples

I hope you can configure the generator to your needs! If you find bugs or anything missing, then please add an issue to the GitHub repository.

If you want to ask questions or need help, then I suggest you hop on my Odin Discord server: https://discord.gg/4FsHgtBmFK

## Thanks for reading!

If you liked this article, then perhaps you'd like my book "Understanding the Odin Programming Language": https://odinbook.com

Sign up for my newsletter for a monthly summary of everything I've been up to: https://karlzylinski.substack.com.

You can support my blog, my YouTube channel and my open-source projects by [becoming a patron](https://patreon.com/karl_zylinski).

Have a nice day!
/Karl Zylinski