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

### Update 2025

> [!NOTE]
> Use cmd for all commands NOT powershell and make sure you ran vcvarsall!

Download the depot_tools as a subfolder into this folder manually and put it in the PATH and at least run `gclient` once (as described in the depot_tools manual). This is mainly so that we get a runnable Python 3 before calling `build.py`. A standalone Python3 installation and using the builtin auto download in `build.py` did not work properly.

> [!NOTE]
> To disable downloading/syncing V8 on every build invocation comment that specific subprocess call in `build.py`.

The vanilla build script with the following invocation works perfectly fine for V8 13.3 and produces a DLL build of V8 packaged in a nuget package.

```bat
python3 build.py --platform=x64 --libs=shared --version=13.3
```

However, this DLL build is not linkable with MSVC due to differing stdlibs (at least that's how I understand it). This linker error is reproducable with a minimal MSVC project having only the V8 `hello_world.cc` from `v8/samples`.

```txt
error LNK2019: unresolved external symbol "class std::unique_ptr<class v8::Platform,struct std::default_delete<class v8::Platform> > __cdecl v8::platform::NewDefaultPlatform(int,enum v8::platform::IdleTaskSupport,enum v8::platform::InProcessStackDumping,class std::unique_ptr<class v8::TracingController,struct std::default_delete<class v8::TracingController> >,enum v8::platform::PriorityMode)" (?NewDefaultPlatform@platform@v8@@YA?AV?$unique_ptr@VPlatform@v8@@U?$default_delete@VPlatform@v8@@@std@@@std@@HW4IdleTaskSupport@12@W4InProcessStackDumping@12@V?$unique_ptr@VTracingController@v8@@U?$default_delete@VTracingController@v8@@@std@@@4@W4PriorityMode@12@@Z) referenced in function main
probe.exe : fatal error LNK1120: 1 unresolved externals
```

> [!NOTE]
> For the MSVC test project you must set the C++ standard to 20, however, this is not enough because MSVC doesn't report the C++ version correctly to the preprocessor doing just that, you have to
> also add the additional compiler flag `Zc:__cplusplus` manually otherwise the V8 headers will complain.

Disabling the custom V8 stdlib with `use_custom_libcxx : False` does not work and leads to linker problems in the abseil thirdparty dependency preventing a build of V8 in the first place. What worked is a monolith (that is statically linked) build of V8 that disables the custom stdlib. But that means we also have to then statically link with V8 which immediately disqualifies the `/MD` flag which then means we can't use `/clr`. So the only way forward with a monolith build, would be to wrap all necessary functionality in plain C calls and build a standalone statically linked wrapper DLL we then consume from either Javascript.Net or directly from C#...

```bat
python3 build.py --platform=x64 --libs=monolith --version=13.3
```

Google currently makes it impossible to use a DLL build without their custom stdlib and furthermore deprecated the MSVC build with V8 13, and plans to remove the `use_custom_libcxx` flag later in 2025.

The monolith build without the custom stdlib only worked with the following `GN_OPTIONS` in `build.py` as well as adding `"-D_SILENCE_CXX20_OLD_SHARED_PTR_ATOMIC_SUPPORT_DEPRECATION_WARNING"` to the `cflags` array in `config("toolchain")` in `v8/BUILD.gn`. The other commented flags were experiments/taken from the ClearScript build script (see link dump below). Disabling pointer compression and SMI optimizations is necessary to make the build work with the custom libc (I did not test these two options individually though, so it might be that one of them suffices).

NOT GOOD ENOUGH YET!!!!
```py
GN_OPTIONS = {
	'use_custom_libcxx' : False,
	# 'use_custom_libcxx_for_host' : False, # probably unnecessary
	'fatal_linker_warnings': False,
	# 'use_thin_lto' : False,
	# 'v8_embedder_string' : "-Foo",
	'v8_enable_pointer_compression' : False,
	'v8_enable_31bit_smis_on_64bit_arch' : False,
	'v8_use_external_startup_data' : False,
}
```

Unordered link dump:

- https://groups.google.com/g/v8-users/c/J8Q6VrX9e4M/m/DVJYVq8MAwAJ
- https://www.mail-archive.com/v8-users@googlegroups.com/msg14711.html
- https://groups.google.com/g/v8-users/c/BIGcXBVMPR8
- https://issues.chromium.org/issues/41496756
- https://github.com/microsoft/ClearScript/blob/master/V8Update.cmd (ClearScript V8 wrapper build script, they are currently also stuck on 12.3 and also link statically)
- https://gist.github.com/jhalon/5cbaab99dccadbf8e783921358020159
- https://issues.chromium.org/issues/40148176

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
    * `v143` - for Visual Studio 2022
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

  * Visual C++ toolset (version >=2022)
  * Python 3.x
  * Git >= 1.9
  * NuGet (https://dist.nuget.org/index.html)

To build V8 and make NuGet packages:

  1. Run `build.py` with optional command-line arguments.
  2. Publish `nuget/*.nupkg` files after successful build.

Build script `build.py` supports command-line arguments to specify package build options:

  1. V8 version branch/tag name `--version`, default is `lkgr` branch (last known good revision)
  2. Platform `--platform`, default are both [`x86`, `x64`]
  3. Configuration `--config`, default are both [`Debug`, `Release`]
  4. Libraries kind `--libs`, default are both [`shared`, `monolith`] (i.e. dll and static libs)
  5. Additional V8 gn options `--gn-option` in key=value format
  6. Print all available options with `--help` switch

For example, to build V8 version 12.8 for x64 dlls, both debug and release run it as:

```
python3 build.py --version=12.8 --platform=x64 --libs=shared
```
