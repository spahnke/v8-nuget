# NuGet package for V8 JavaScript Engine

## Building with the script in this repository

Follow the first two steps to install depot tools, VS 2022, and the Windows SDK under [Building V8 manually](#building-v8-manually). As of this writing V8
requires the Windows SDK 10.0.22621.0 with debugging tools to build. Then run the following command to build the nuget packages locally in the specified version
from a Visual Studio Developer cmd/pwsh (this is important in order for several VC++ specific environment variables to be set).

```powershell
python3 .\build.py --platform=x64 --libs=shared --use-clang --version=11.9
```

> [!NOTE]
> To build the latest version (aka LKGR) run without the version flag

> [!WARNING]
> As of April 2024 this results in link errors for abseil-cpp.

## Building V8 manually

> [!NOTE]
> This is information as of April 2023

- Get and init depot tools and add them to your PATH [^1] (the depot tools include Python2 and Python3, as well as several other build tools)
- Install VS 2022 with C++ tools (143) and the Windows SDK (10.0.22621.0) including the debug tools (this must be installed **standalone** from [here](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/) not through the VS installer; be sure to just install the debug tools) [^2]
- Set the `DEPOT_TOOLS_WIN_TOOLCHAIN=0` environment variable [^2]
- Fetch the V8 source code [^3]

  ```sh
  fetch --no-history v8
  cd v8
  ```

- Navigate into the `v8` directory and build according to [^2] [^4] and [^5]

  ```sh
  gclient sync
  python3 tools/dev/v8gen.py x64.release # generate build files
  gn args out.gn\x64.release # open args file in editor or supply parameters directly (next line doesn't work yet...):
  # gn gen out.gn\x64.release --args='treat_warnings_as_errors=false fatal_linker_warnings=false v8_enable_fast_torque=false v8_enable_verify_heap=false v8_use_external_startup_data=false v8_enable_sandbox=false use_custom_libcxx=false is_debug=false enable_iterator_debugging=false target_cpu="x64" is_clang=true is_component_build=true v8_monolithic=false'
  ninja -C out.gn\x64.release # compile
  ```

  With the following parameters [^6]

  ```ini
  treat_warnings_as_errors = false
  fatal_linker_warnings = false
  v8_enable_fast_torque = false
  v8_enable_verify_heap = false
  v8_use_external_startup_data = false
  v8_enable_sandbox = false
  use_custom_libcxx = false
  is_debug = false
  enable_iterator_debugging = false
  target_cpu = "x64"
  is_clang = true
  is_component_build = true
  v8_monolithic = false
  ```

[^1]: <https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up>
[^2]: <https://medium.com/angular-in-depth/how-to-build-v8-on-windows-and-not-go-mad-6347c69aacd4>
[^3]: <https://v8.dev/docs/source-code>
[^4]: <https://v8.dev/docs/build>
[^5]: <https://v8.dev/docs/build-gn#manual>
[^6]: <https://www.chromium.org/developers/gn-build-configuration/>

## Information

This packages contain prebuilt V8 binaries, debug symbols, headers and
libraries required to embed the V8 JavaScript engine into a C++ project.

| Package                     | Version
|-----------------------------|----------------------------------------------------------------------------------------------------------------------|
|V8 x64 for Visual Studio 2022|[![NuGet](https://img.shields.io/nuget/v/v8-v143-x64.svg)](https://www.nuget.org/packages/v8-v143-x64/)|
|V8 x86 for Visual Studio 2022|[![NuGet](https://img.shields.io/nuget/v/v8-v143-x86.svg)](https://www.nuget.org/packages/v8-v143-x86/)|
|V8 x64 for Visual Studio 2019|[![NuGet](https://img.shields.io/nuget/v/v8-v142-x64.svg)](https://www.nuget.org/packages/v8-v142-x64/)|
|V8 x86 for Visual Studio 2019|[![NuGet](https://img.shields.io/nuget/v/v8-v142-x86.svg)](https://www.nuget.org/packages/v8-v142-x86/)|
|V8 x64 for Visual Studio 2017|[![NuGet](https://img.shields.io/nuget/v/v8-v141-x64.svg)](https://www.nuget.org/packages/v8-v141-x64/)|
|V8 x86 for Visual Studio 2017|[![NuGet](https://img.shields.io/nuget/v/v8-v141-x86.svg)](https://www.nuget.org/packages/v8-v141-x86/)|
|V8 x64 for Visual Studio 2015|[![NuGet](https://img.shields.io/nuget/v/v8-v140-x64.svg)](https://www.nuget.org/packages/v8-v140-x64/)|
|V8 x86 for Visual Studio 2015|[![NuGet](https://img.shields.io/nuget/v/v8-v140-x86.svg)](https://www.nuget.org/packages/v8-v140-x86/)|
|V8 x64 for Visual Studio 2013|[![NuGet](https://img.shields.io/nuget/v/v8-v120-x64.svg)](https://www.nuget.org/packages/v8-v120-x64/)|
|V8 x86 for Visual Studio 2013|[![NuGet](https://img.shields.io/nuget/v/v8-v120-x86.svg)](https://www.nuget.org/packages/v8-v120-x86/)|
|V8 x64 for Visual Studio 2017 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v141_xp-x64.svg)](https://www.nuget.org/packages/v8-v141_xp-x64/)|
|V8 x86 for Visual Studio 2017 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v141_xp-x86.svg)](https://www.nuget.org/packages/v8-v141_xp-x86/)|
|V8 x64 for Visual Studio 2015 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v140_xp-x64.svg)](https://www.nuget.org/packages/v8-v140_xp-x64/)|
|V8 x86 for Visual Studio 2015 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v140_xp-x86.svg)](https://www.nuget.org/packages/v8-v140_xp-x86/)|
|V8 x64 for Visual Studio 2013 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v120_xp-x64.svg)](https://www.nuget.org/packages/v8-v120_xp-x64/)|
|V8 x86 for Visual Studio 2013 XP platform toolset|[![NuGet](https://img.shields.io/nuget/v/v8-v120_xp-x86.svg)](https://www.nuget.org/packages/v8-v120_xp-x86/)|


## Usage

To use V8 in a project install the package `v8-$PlatformToolset-$Platform.$Version`
from a console with `nuget install` command or from inside of Visual Studio
(see menu option *Tools -> NuGet Package Manager -> Manage NuGet Packages for Solution...*)
where

  * `$PlatformToolset` is the C++ toolset version used in Visual Studio:
    * `v120` - for Visual Studio 2013
    * `v140` - for Visual Studio 2015
    * `v141` - for Visual Studio 2017
    * `v142` - for Visual Studio 2019
    * `v120_xp` - for Visual Studio 2013 XP platform toolset
    * `v140_xp` - for Visual Studio 2015 XP platform toolset
    * `v141_xp` - for Visual Studio 2017 XP platform toolset
  
  * `$Platform` is a target platform type, currently `x86` or `x64`.

  * `$Version` is the actual V8 version, one of https://chromium.googlesource.com/v8/v8.git/+refs

There are 3 package kinds:

  * `v8-$PlatformToolset-$Platform.$Version` - contains developer header and 
    library files; depends on `v8.redist` package

  * `v8.redist-$PlatformToolset-$Platform.$Version` - prebuilt V8 binaries:
    dlls, blobs, etc.

  * `v8.symbols-$PlatformToolset-$Platform.$Version` - debug symbols for V8:
    [pdb files](https://en.wikipedia.org/wiki/Program_database)

After successful packages installation add `#include <v8.h>` in a C++  project
and build it. All necessary files (*.lib, *.dll, *.pdb) would be referenced
in the project automatically with MsBuild property sheets.


## How to build

This section is mostly for the package maintainers who wants to update V8.

Tools required to build V8 NuGet package on Windows:

  * Visual C++ toolset (version >=2013)
  * Python 2.X
  * Git
  * NuGet (https://dist.nuget.org/index.html)

To build V8 and make NuGet packages:

  1. Run `build.py` with optional command-line arguments.
  2. Publish `nuget/*.nupkg` files after successful build.
  
Build script `build.py` supports command-line arguments to specify package build options:

  1. V8 version branch/tag name (or `V8_VERSION` environment variable), default is `lkgr` branch
  2. Target platform (or `PLATFORM` evnironment variable), default is [`x86`, `x64`]
  3. Configuration (or `CONFIGURATION` environment variable), default is [`Debug`, `Release`]
  4. XP platofrm toolset usage flag (or `XP` environment variable), default is not set
