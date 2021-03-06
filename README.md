[![Build Status](https://travis-ci.org/Mizux/cmake-swig.svg?branch=master)](https://travis-ci.org/Mizux/cmake-swig)
[![Build status](https://ci.appveyor.com/api/projects/status/a8pir5oh0gpt2q5u/branch/master?svg=true)](https://ci.appveyor.com/project/Mizux/cmake-swig/branch/master)

# Introduction

This is an example of how to create a Modern [CMake](https://cmake.org/) C++ Project
with the [SWIG](http://www.swig.org) code generator.  

This project should run on Linux, Mac and Windows.

note: Currently, only Python wrapper is tested (under Linux).  

roadmap: add java and csharp SWIG and test under Mac and Windows also.

# CMake Dependencies Tree
To complexify a little, the CMake project is composed of three libraries (Foo, Bar and FooBar)
with the following dependencies:  
```sh
Foo:
Bar:
FooBar: PRIVATE Foo Bar
```
## Project directory layout
Thus the project layout is as follow:
```sh
 CMakeLists.txt // meta CMake doing the orchestration and python packaging
 Foo
 ├── CMakeLists.txt
 ├── include
 │   └── foo
 │       └── Foo.hpp
 ├── python
 │   ├── CMakeLists.txt
 │   └── foo.i
 └── src
     └── Foo.cpp
 Bar
 ├── CMakeLists.txt
 ├── include
 │   └── bar
 │       └── Bar.hpp
 ├── python
 │   ├── bar.i
 │   └── CMakeLists.txt
 └── src
     └── Bar.cpp
 FooBar
 ├── CMakeLists.txt
 ├── include
 │   └── foobar
 │       └── FooBar.hpp
 ├── python
 │   ├── CMakeLists.txt
 │   └── foobar.i
 └── src
     ├── FooBar.cpp
     └── main.cpp
```

# C++ Project Build
To build the C++ project, as usual:
```sh
mkdir build && cd build
cmake ..
make
```
note: SWIG automatically put its target(s) in `all`, thus `make` will also call
swig and generate `_module.so`.

# Python
## Build directory layout
Since we want to use the [CMAKE_BINARY_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_BINARY_DIR.html) to generate the python binary package.  
We want this layout (tree build --prune -P "*.py|*.so"):
```sh
 Bar
 ├── __init__.py
 ├── libBar.so
 ├── pyBar.py
 └── _pyBar.so
 Foo
 ├── __init__.py
 ├── libFoo.so
 ├── pyFoo.py
 └── _pyFoo.so
 FooBar
 ├── __init__.py
 ├── libFooBar.so
 ├── pyFooBar.py
 └── _pyFooBar.so
```

## Build the Binary Package
To build the python package, simply run:
```sh
make bdist
```

## Tips
While running swig to generate Python wrapper it's easy thanks to
[UseSWIG](https://cmake.org/cmake/help/latest/module/UseSWIG.html) module.  

Creating a Python binary package containing all `.py` and `.so` (with good rpath) is not so easy... 

### Managing SWIG generated files
Since python use the directory name where `__init__.py` file is located.  
We would like to have `pyFoo.py` generated file in `build/Foo` and not in `build/Foo/python`.  
You can use `CMAKE_SWIG_DIR` to change the output directory for the `.py` file e.g.:
```cmake
set(CMAKE_SWIG_OUTDIR ${CMAKE_CURRENT_BINARY_DIR}/..)
```
And you can use `CMAKE_LIBRARY_OUTPUT_DIRECTORY` to change the output directory for the `.so` file e.g.:
```cmake
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/..)
```
[optional]You can use `SWIG_OUTFILE_DIR` to change the output directory for the `.cxx` file e.g.:
```cmake
set(SWIG_OUTFILE_DIR ${CMAKE_CURRENT_BINARY_DIR}/..)
```
Then you only need to create a `__init__.py` file in `build/Foo` to be able to use
the build directory to generate the Python package.

### Managing RPATH
Since we want to use the [CMAKE_BINARY_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_BINARY_DIR.html) to generate the python binary package.
We need to enable:
```cmake
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
```
And have a finely tailored rpath for each library.

For Foo/CMakeLists.txt (which depend on nothing):
```cmake
set(CMAKE_INSTALL_RPATH "$ORIGIN/")
```

For FooBar/CMakeLists.txt (which depend on Foo & Bar):
```cmake
set(CMAKE_INSTALL_RPATH "$ORIGIN/../Foo:$ORIGIN/../Bar:$ORIGIN/")
```

note: you allways need `$ORIGIN/` since `_pyFoo.so` will depend on `libFoo.so`
(which will be built in the same directory see above).

### Why setup.py has to be generated
To avoid to put hardcoded path to SWIG `.so` generated files,
we could use `$<TARGET_FILE:tgt>` to retrieve the file (and also deal with Mac/Windows suffix, and target dependencies).  
In order for setup.py to use
[cmake generator expression](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#informational-expressions)
(e.g. $<TARGET_FILE:_pyFoo>). We need to generate it at build time (e.g. using
[add_custom_command()](https://cmake.org/cmake/help/latest/command/add_custom_command.html)).  
note: This will also add automatically a dependency between the command and the TARGET !

## Testing Python
### Testing using virtualenv

### Testing using the `CMAKE_BINARY_DIR`

# Contributing

The [CONTRIBUTING.md](./CONTRIBUTING.md) file contains instructions on how to
file the Contributor License Agreement before sending any pull requests (PRs).
Of course, if you're new to the project, it's usually best to discuss any
proposals and reach consensus before sending your first PR.

# License

Apache 2. See the LICENSE file for details.

# Disclaimer

This is not an official Google product, it is just code that happens to be
owned by Google.

