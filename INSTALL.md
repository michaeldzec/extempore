# Building Extempore from Source

## Build depenencies

You'll need

a **C++ compiler toolchain**, e.g.

- `sudo apt-get install g++` on Ubuntu/Debian
- `sudo yum install gcc gcc-c++` on Fedora/CentOS/RHEL
- Xcode or the [command line tools](https://developer.apple.com/library/ios/technotes/tn2339/_index.html#//apple_ref/doc/uid/DTS40014588-CH1-WHAT_IS_THE_COMMAND_LINE_TOOLS_PACKAGE_) on OSX
- Visual Studio on Windows (the
  [Community 2015](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)
  version is now free)

**git**

- `sudo apt-get install git` on Ubuntu/Debian
- `sudo yum install git` on Fedora/CentOS/RHEL
- `brew install git` on OSX with Homebrew
- `choco install git` on Windows with Chocolatey

**CMake** (version 3.1 or greater)

- `sudo yum install cmake` on Fedora/CentOS/RHEL
- `brew install cmake` on OSX with Homebrew
- `choco install cmake` on Windows with Chocolatey

The Ubuntu 15.04 package archive only includes CMake v3.0, but you can
get a more up-to-date version through a package archive

```
sudo apt-get install software-properties-common && sudo add-apt-repository ppa:george-edison55/cmake-3.x && sudo apt-get update && sudo apt-get install cmake
```

### ALSA (Linux only)

To use the ALSA portaudio backend (which is probably what you want,
unless you have a real reason to go with something else) you'll need
the libasound package at build-time, e.g. (on Ubuntu)

```
sudo apt-get install libasound2-dev
```

If you really want to use a different backend (e.g. `jack`) then you
can hack the `PA_USE_*` definitions in `CMakeLists.txt`

### Boost (Windows only)

We still need one component of the **Boost** libs on Windows
(specifically the ASIO component for TCP/UDP handling). If you've got
the NuGet command line client installed, you can probably do

```
nuget install boost-vc140 & nuget install boost_system-vc140 & nuget install boost_regex-vc140 & nuget install boost_date_time-vc140
```

It doesn't matter how you get these deps, but the Extempore CMake
build process expects to find them in `libs/win64/lib` (for the
library files) and `libs/win64/include` (for the headers). So you'll
need to (probably manually) move them in there, which is a bit
painful, but it's a one-time process.

### LLVM 3.7

Grab the
[3.7.0 source tarball](http://llvm.org/releases/download.html#3.7.0),
apply the `extempore-llvm-3.7.0.patch` in `extras/`

```
cd /path/to/llvm-3.7.0.src
patch -p0 < /path/to/extempore/extras/extempore-llvm-3.7.0.patch
```

Then build LLVM, moving the libraries into `/path/to/extempore/llvm`
as part of the `install` step

```
mkdir cmake-build && cd cmake-build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_TERMINFO=OFF -DLLVM_ENABLE_ZLIB=OFF -DCMAKE_INSTALL_PREFIX=/path/to/extempore/llvm .. && make && make install
```

On **Windows**, you'll also need to specify a 64-bit generator e.g.
`-G"Visual Studio 14 2015 Win64"`

## Building Extempore

### Generate build files with CMake

In your `extempore` directory,

```
mkdir cmake-build && cd cmake-build
cmake -DEXT_LLVM_DIR=/path/to/extempore/llvm ..
```

If you've set the `EXT_LLVM_DIR` environment variable you don't have
to provide it again to CMake.

On **Windows**, you'll need to give CMake a few more details

```
md cmake-build && cd cmake-build
cmake -G"Visual Studio 14 2015 Win64" -DEXT_LLVM_DIR=c:\path\to\extempore\llvm ..
```

### Build Extempore

On **Linux/OSX** CMake will generate a `Makefile` in `cmake-build`,
with a few useful targets:

- `make` will build Extempore
- `make install` will install `extempore` into `/usr/local/bin` and
  the rest of the files (the "Extempore share directory") into
  `/usr/local/share/extempore`
- `make uninstall` will remove the installed files
- `make aod_stdlibs` will ahead-of-time compile the standard library

On **Windows**, CMake will generate a Visual Studio solution (`.sln`)
in `cmake-build`. Open it, and build the `extempore` target.

## Building the Extempore standard library

### Linux/OSX

It's pretty straightforward. You should be able to get most things
through your package manager.

### Windows

In the shell where you run `extempore.exe`, you'll need to set the
build vars so you can call `link` during the AOT-compilation process.
To do this, there's a helpful `.bat` file in `extras/`
```
.\extras\ms_build_vars.bat
```
You may also need to add Extempore's Windows lib directory
`libs/win64/lib` to your `PATH` environment variable.  There are a few
ways to do this, but the easiest is probably just (replacing my path
to Extempore with wherever it is on your machine)
```
set PATH=C:/Users/ben/Code/extempore/libs/win64/lib;%PATH%
```

#### libsndfile

Just grab the Windows 64-bit installer from
(http://www.mega-nerd.com/libsndfile/), and copy `libsndfile-1.dll`
and `libsndfile-1.lib` into `extempore/libs/Win64/lib`

#### GLEW

We don't use GLEW directly, but we use it to build nanovg.

http://glew.sourceforge.net/

Download the latest stable version (I used 1.13.0).

```
mkdir cmake-build && cd cmake-build
cmake -G"Visual Studio 14 2015 Win64" ../build/cmake/
```

nanovg requires the `include/GL/glew.h` header and the `libglew32.lib`
"static" lib.

To use the shared library version of GLEW, you need to copy the
headers and libraries into their destination directories. On Windows
this typically boils down to copying:

```
bin/glew32.dll	        to     	%SystemRoot%/system32
lib/glew32.lib	        to     	{VC Root}/Lib
include/GL/glew.h	    to     	{VC Root}/Include/GL
include/GL/wglew.h	    to     	{VC Root}/Include/GL
```

#### GLFW

```
cmake -DBUILD_SHARED_LIBS=ON -DGLFW_BUILD_EXAMPLES=OFF -DGLFW_BUILD_TESTS=OFF ..
```

#### stb_image

```
git clone git@github.com:benswift/stb
cd stb && mkdir cmake-build && cd cmake-build
cmake -G"Visual Studio 14 2015 Win64" ..
```

#### nanovg

```
git clone git@github.com:benswift/nanovg
cd nanovg && mkdir cmake-build && cd cmake-build
cmake -G"Visual Studio 14 2015 Win64"
```