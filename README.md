# Ultralight

__Welcome to the Ultralight Universal SDK!__

This package contains everything you need to start building cross-platform HTML apps.

## Useful Links

| Link              | URL                                 |
| ----------------- | ----------------------------------- |
| __Slack Channel__ | <https://chat.ultralig.ht>          |
| __Support Forum__ | <https://discuss.ultralig.ht>       |
| __Twitter__       | <https://twitter.com/ultralight-ux> |


# Table of Contents

 - [Getting Started](#getting-started)
 	- [Build Requirements](#build-requirements)
 		- [Windows Build Requirements](#windows-build-requirements)
 		- [macOS Build Requirements](#macos-build-requirements)
 		- [Linux Build Requirements](#linux-build-requirements)
 - [Building Sample Projects](#building-sample-projects)
 	- [Building Samples with CMake (All Platforms)](#building-samples-with-cmake-all-platforms)
 	- [Building Samples (Windows)](#building-samples-windows)
 	- [Building Samples (macOS)](#building-samples-macos)
 - [Using the C++ API](#using-the-c-api)
 	- [Compiler / Linker Flags](#compiler-linker-flags)
 		- [Disabling RTTI](#disabling-rtti)
 		- [Setting Your Include Directories](#setting-your-include-directories)
 		- [Linking to the Library (Windows / MSVC)](#linking-to-the-library-windows-msvc)
 		- [Linking to the Library (Linux)](#linking-to-the-library-linux)
 		- [Linking to the Library (macOS)](#linking-to-the-library-macos)
 	- [API Headers](#api-headers)
 	- [Platform Handlers](#platform-handlers)
 		- [Config](#config)
 		- [GPUDriver](#gpudriver)
 		- [FontLoader](#fontloader)
 		- [FileSystem](#filesystem)
  	- [Rendering]()
 		- [Creating the Renderer]()
 		- [Updating the Renderer]()
 		- [Drawing View Overlays]()
 	- [Managing Views]()
 		- [Creating Views]()
 		- [Loading Content]()
 		- [Passing Mouse / Keyboard Input]()
 		- [Handling Events]()
 	- [JavaScript Integration]()
		- [JavaScriptCore API]()
		- [Obtaining a JSContext]()
		- [DOMReady Event]()
		- [Evaluating Scripts]()
		- [Calling JS Functions from C++]()
		- [Calling C++ Functions from JS]()
 - [Using the Framework Code]()
 	- [Select Framework Platform]()
	- [Writing Your Application]()
	- [JSHelpers Utility Code]()

# Getting Started

Before you get started, you will need the following on each platform:

## Build Requirements

All platforms include an __OpenGL-based sample (powered by GLFW)__. To build these cross-platform CMake/GLFW samples you will need:

 - CMake 2.8.12 or later
 - OpenGL 3.2 or later
 - Compiler with C++11 or later

### Windows Build Requirements

 - In addition to the above...
 	- Visual Studio 2015 or later
 	- DirectX 11+ or later __[optional], only for D3D11-based Samples__
 		* _Note: The DirectX SDK is included with the Windows SDK in Windows 8+_

### macOS Build Requirements
 - In addition to the above...
 	- XCode 8.0+ or later
 	- Metal 2 (macOS High Sierra or later) __[optional], only for Metal-based Samples__

### Linux Build Requirements

 - In addition to the above...
 	- 64-bit Debian-based OS (__Debian 9.5.0+__ OR __Ubuntu 12.04.5+__)


# Building Sample Projects

## Building Samples with CMake (All Platforms)

To generate projects for your platform, run the following from this directory:

```
mkdir build
cd build
cmake ..
```

On __macOS__ and __Linux__ you can then simply build the generated Makefile:

```
make
```

On __Windows__, a Visual Studio solution will be generated in the `build` directory which you can then open and build.

```
Ultralight.sln
```

### Running the Samples

On __macOS__ and __Linux__ the projects will be built to:

```
/build/samples/Browser/
```

On __Windows__ the projects will be built to:

```
/build/samples/Browser/$(Configuration)
```

## Building Samples (Windows Only)

A D3D11-based Browser sample is included on Windows, open the following solution in Visual Studio to build and run it:

```
/samples/Browser/projects/win/Browser.sln
```

## Building Samples (macOS Only)

A Metal-based Browser sample is included on macOS, open the following project in XCode to build and run it:

```
/samples/Browser/projects/mac/Browser.xcodeproj
```

# Using the C++ API

## Compiler / Linker Flags

### Disabling RTTI

Ultralight is built __without Run-Time Type Information (RTTI)__, you will need to disable RTTI in your C++ flags to link successfully to the library.

Check your compiler's documentation on how to disable RTTI, on __MSVC__ the command-line flag is `/GR-`, and on __GCC/Clang__ it is `-fno-rtti`. Here's a CMake script that sets these automatically per-platform:

```
if (MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /GR-")
elseif (UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
endif()
```

### Setting your Include Directories

To use Ultralight in your C++ code, simply add the following directory to your project's include directories (replace `$(ULTRLIGHT_SDK_ROOT)` with the actual path you've placed the SDK):

```
$(ULTRALIGHT_SDK_ROOT)/include/
```

If you will be using any of the framework code (additional code that helps set up platform-specific windows, GPU contexts/drivers, file systems, etc.), you should also include the following: 

```
$(ULTRALIGHT_SDK_ROOT)/deps/
```

### Linking to the Library (Windows / MSVC)

In Visual Studio, go to __Linker &rarr; General &rarr; Additional Library Directories__ in your project's properties and set one of the following:

> For __Win32 / x86__ Platforms:
> 
> ```
> $(ULTRALIGHT_SDK_ROOT)/lib/win/x86/
> ```

__OR__

> For __x64__ Platforms:
>
> ```
> $(ULTRALIGHT_SDK_ROOT)/lib/win/x64/
> ```

Then, go to __Linker &rarr; Input &rarr; Additional Dependencies__ and add the following:

```
Ultralight.lib
UltralightCore.lib
WebCore.lib
```

### Linking to the Library (Linux)

First, copy the shared libraries in `$(ULTRALIGHT_SDK_ROOT)/bin/linux` to your OS's standard library directory.

Then, add the following to your Makefile's `LDFLAGS`:

```
-lUltralight -lUltralightCore -lWebCore
```

### Linking to the Library (macOS)

Within XCode, select your target and go to __General &rarr; Linked Frameworks and Libraries__ and add the following:

```
Ultralight.framework
WebCore.framework
UltralightCore.dylib
```

## API Headers

Simply include `<Ultralight/Ultralight.h>` at the top of your code to import the API.

```cpp
#include <Ultralight/Ultralight.h>
```

Ultralight also exposes the full __JavaScriptCore__ API so that users can make native calls to/from the JavaScript VM. To include these headers simply add:

```cpp
#include <JavaScriptCore/JavaScriptCore.h>
```


## Platform Handlers

Most OS-specific tasks in Ultralight have been left up to the user to provide via a virtual platform interface. This keeps the codebase small and grants users greater control over the behavior of the library.

At a minimum, users are expected to provide a `Config`, `GPUDriver`, and `FontLoader` before creating the `Renderer`.

```cpp
auto& platform = Platform::instance();

platform.set_config(config_);
platform.set_gpu_driver(gpu_driver_);
platform.set_file_system(file_system_);
platform.set_font_loader(font_loader_);
```

### Config

The __Config__ class allows you to configure renderer behavior and runtime WebKit options.

The most common things to customize are `face_winding` for front-facing triangles and `device_scale_hint` for application DPI scaling (used for oversampling raster output).

You can also set the default font (instead of Times New Roman).

```cpp
Config config_;
config.face_winding = kFaceWinding_Clockwise; // CW in D3D, CCW in OGL
config.device_scale_hint = 1.0;               // Set DPI to monitor DPI scale
config.font_family_standard = "Segoe UI";     // Default font family

Platform::instance().set_config(config_);
```

### GPUDriver

The virtual __GPUDriver__ interface is used to perform all rendering in Ultralight.

Reference implementations for Direct3D11, OpenGL 3.2, Metal 2, and others are provided in the Framework code (see `deps/Framework/platform/` in the SDK).

```cpp
class UExport GPUDriver {
public:
  virtual ~GPUDriver();

  /******************************
   * Synchronization            *
   ******************************/

  virtual void BeginSynchronize() = 0;
  virtual void EndSynchronize() = 0;

  /******************************
   * Textures                   *
   ******************************/

  virtual uint32_t NextTextureId() = 0;
  virtual void CreateTexture(uint32_t texture_id,
                             Ref<Bitmap> bitmap) = 0;
  virtual void UpdateTexture(uint32_t texture_id,
                             Ref<Bitmap> bitmap) = 0;
  virtual void BindTexture(uint8_t texture_unit,
                           uint32_t texture_id) = 0;
  virtual void DestroyTexture(uint32_t texture_id) = 0;

  /******************************
   * Render Buffers             *
   ******************************/

  virtual uint32_t NextRenderBufferId() = 0;
  virtual void CreateRenderBuffer(uint32_t render_buffer_id,
                                 const RenderBuffer& buffer) = 0;
  virtual void BindRenderBuffer(uint32_t render_buffer_id) = 0;
  virtual void SetRenderBufferViewport(uint32_t render_buffer_id,
                                       uint32_t width,
                                       uint32_t height) = 0;
  virtual void ClearRenderBuffer(uint32_t render_buffer_id) = 0;
  virtual void DestroyRenderBuffer(uint32_t render_buffer_id) = 0;

  /******************************
   * Geometry                   *
   ******************************/

  virtual uint32_t NextGeometryId() = 0;
  virtual void CreateGeometry(uint32_t geometry_id,
                              const VertexBuffer& vertices,
                              const IndexBuffer& indices) = 0;
  virtual void UpdateGeometry(uint32_t geometry_id,
                              const VertexBuffer& vertices,
                              const IndexBuffer& indices) = 0;
  virtual void DrawGeometry(uint32_t geometry_id,
                            uint32_t indices_count,
                            uint32_t indices_offset,
                            const GPUState& state) = 0;
  virtual void DestroyGeometry(uint32_t geometry_id) = 0;

  /******************************
   * Command List               *
   ******************************/

  virtual void UpdateCommandList(const CommandList& list) = 0;
  virtual bool HasCommandsPending() = 0;
  virtual void DrawCommandList() = 0;
};
```

### FontLoader

The virtual __FontLoader__ interface is used to load font files (TrueType and OpenType files, usually stored somewhere in the Operating System) by font family.

Reference implementations for Windows and macOS, as well as a simple loader that simply returns an embedded Roboto TTF for all queries (`FontLoaderRoboto`) is provided in the Framework code (see `deps/Framework/platform/` in the SDK).

```cpp
class UExport FontLoader {
public:
  virtual ~FontLoader();

  virtual String16 fallback_font() const = 0;
  
  virtual Ref<Buffer> Load(const String16& family,
                           int weight,
                           bool italic,
                           float size) = 0;
};
```

### FileSystem

The virtual __FileSystem__ interface is used for loading File URLs (eg, `file:///page.html`) and the JavaScript FileSystem API.

This API should be used to load any HTML/JS/CSS assets you've bundled with your application.

Reference implementations for Windows and macOS, as well as a basic FileSystem loader that uses the C++ STL (`FileSystemBasic`) is provided in the Framework code  (see `deps/Framework/platform/` in the SDK).

Only a small subset needs to be implemented to support File URL loading, specifically the following: 

```
class UExport FileSystem {
public:
  virtual ~FileSystem();

  virtual bool FileExists(const String16& path) = 0;

  virtual bool GetFileSize(FileHandle handle,
                           int64_t& result) = 0;

  virtual bool GetFileMimeType(const String16& path,
                               String16& result) = 0;

  virtual FileHandle OpenFile(const String16& path,
                              bool open_for_writing) = 0;

  virtual void CloseFile(FileHandle& handle) = 0;

  virtual int64_t ReadFromFile(FileHandle handle,
                               char* data,
                               int64_t length) = 0;
};
```

## Rendering

### Creating the Renderer

The __Renderer__ class is used to create __Views__ and update them.

You should create only one instance per application lifetime:

```cpp
Ref<Renderer> renderer = Renderer::Create();
```

### Updating the Renderer

Once per frame, you should call `Renderer::Update()` and `Renderer::Render()`, here is brief outline of how your Update function should look:

```cpp
void MyApplication::Update() {
  // Update internal logic (timers, event callbacks, etc.)
  renderer->Update();

  driver->BeginSynchronize();

  // Render all active views to command lists and dispatch calls to GPUDriver
  renderer->Render();

  driver->EndSychronize();

  // Draw any pending commands to screen
  if (driver->HasCommandsPending()) {
    driver->DrawCommandList();
    
    // Perform any additional drawing (Overlays) here...
    DrawOverlays();

    // Flip buffers here.
  }
}
```

### Drawing View Overlays

Views are rendered to an offscreen texture and so it is the user's responsibility to draw this texture to the screen. To get the __Texture ID__ for a __View__, please see `View::render_target()`.

A reference implementation for a View-based Overlay is supplied within the Framework code (see `/deps/Framework/Overlay.h` and `/deps/Framework/Overlay.cpp`).

## Managing Views

__Views__ are used to display and interact with web content in Ultralight.

### Creating Views

To create a View, simply call `Renderer::CreateView()` with your desired width and height:

```
Ref<View> view = renderer_->CreateView(800, 600, false);
```

### Loading Content

You can load content into a View via either `View::LoadHTML()` or `View::LoadURL()`:

```cpp
view->LoadHTML("<h1>Hello World</h1>"); // HTML string

view->LoadURL("http://www.google.com"); // Remote URL

view->LoadURL("file:///asset.html");    // File URL
```
> *__Note__: To load local File URLs, make sure your FileSystem resolves file paths relative to your application's asset directory.*

### Passing Mouse / Keyboard Input

### Handling Events

## JavaScript Integration

### JavaScriptCore API

### Obtaining a JSContext

### DOMReady Event

### Evaluating Scripts

### Calling JS Functions from C++

### Calling C++ Functions from JS

# Using the Framework Code

## Select Framework Platform

## Writing Your Application

```cpp
// Set our Framework platform to GLFW (set this in C Preprocessor Flags)
#define FRAMEWORK_PLATFORM_GLFW 1

#include <Ultralight/Ultralight.h>
#include <Framework/Application.h>
#include <Framework/GPUContext.h>

using namespace ultralight;
using namespace framework;

class MyApplication : public Application {
 public:
  MyApplication(Window& window) {
    gpu_context_ = GPUContext::Create(window, true);
    
    Platform::instance().set_gpu_driver(gpu_context_->driver());
    
    // Perform rest of initialisation here...
  } 
  
  virtual void Update() {
    // Perform update here...
  }
  
 protected:
  std::unique_ptr<GPUContext> gpu_context_;
};

std::unique_ptr<Window> Application::InitWindow(PlatformWindowConfig& config) {
  // Configure window here...
  config.width = 800;
  config.height = 600;
  config.title = "My Application Title";

  return Window::Create(config);
}

std::unique_ptr<Application> Application::Create(Window& window) {
  // Return our custom Application instance
  return std::unique_ptr<Application>(new MyApplication(window));
}
```

## JSHelpers Utility Code
