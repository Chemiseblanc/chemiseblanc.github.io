---
title: My 2024 C++ workflow using Modern CMake and Vcpkg
date: 2024-05-09T23:43:01.723Z
draft: false
longform: true
tags:
  - c++
  - cmake
  - vcpkg
---


Setting up and maintaining a C++ development environment has always been more 
difficult than for newer languages, especially once third party dependencies
are involved, but in the last few years
the ecosystem has improved to the point where it's possible to create a project
that can bootstrap a cross-platform development environment just by opening it
with an editor that supports cmake or with a single command in the terminal.

Vcpkg is an open source dependency management tool developed by microsoft that
integrates into cmake so you can have project-local dependencies detected by the
find_package utility. The recommended usage is to add a git submodule to your
project that points at a release tag of vcpkg and then when configuring cmake,
usind the define CMAKE_TOOLCHAIN_FILE to point to the vcpkg.cmake toolchain.
Some of the weak points I have encountered when following this are:
- Whenever you clone your repo you need to remember to initialize the submodule
   which isn't always done automatically by some IDEs
- If you are already using a custom toolchain file you need to remember to
   set VCPKG_CHAINLOAD_TOOLCHAIN.
- If you want to specify which dependencies should be built as shared or static
   libraries you need to create a custom toolchain file which then complicates
   the former.

I still haven't found a good solution to the last problem but for the first two
I developed a small cmake script [FetchVcpkg.cmake](https://github.com/Chemiseblanc/cmake-utils) which when called at the
beginning of the root CMakeLists.txt fetches or updates vcpkg at configure time.

```cmake
# Assuming FetchVcpkg.cmake has been placed in ${PROJECT_ROOT_DIR}/cmake
cmake_minimum_required(...)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
include(FetchVcpkg)
fetch_vcpkg(TAG 2023.11.20)

project(...)
```
There has previously been discussion about problems with using CMake's
FetchContent to manage Vcpkg which I believe I have solved in this script.
It's also designed to automatically propagate any toolchain
that has been specified by CMAKE_TOOLCHAIN_FILE so it can be used in a drop-in
manner compatible with any existing automation.

Using this tool by itself is usually enough to get development started and ease
third party dependency management but once you need to handle build variants
or want to automate testing and deployment you can end up in the situation
of handwriting shell or CI scripts. Enter CMakePresets.json.

CMake presets are a relativley new way to take the parameters given to cmake
and encode them in a json file that acts as a single source of truth for 
developers, IDEs, and CI/CD systems. Presets are broken up into components
that can be passed as a `--preset=` argument to the various cmake 
sub-commands. Currently presets can be specified for configuration, building, 
testing, packaging, and workflow presets which are used to specify tasks
composed of the previous steps.

Here's an example CMakePresets.json from one of my projects.
```json{
    "version": 6,
    "cmakeMinimumRequired": {
        "major": 3,
        "minor": 25,
        "patch": 0
    },
    "configurePresets": [
        {
            "hidden": true,
            "name": "default",
            "displayName": "Default",
            "binaryDir": "${sourceDir}/artifacts/build/${presetName}",
            "installDir": "${sourceDir}/artifacts/install/${presetName}",
            "cacheVariables": {
                "FETCH_VCPKG_DIR": "${sourceDir}/artifacts/vcpkg",
                "VCPKG_INSTALL_OPTIONS": "--x-abi-tools-use-exact-versions;--clean-after-build"
            }
        },
        {
            "inherits": "default",
            "name": "devel",
            "displayName": "Development",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "BUILD_TESTING": true
            }
        },
        {
            "inherits": "default",
            "name": "release",
            "displayName": "Release",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release",
                "BUILD_TESTING": false
            }
        }
    ],
    "buildPresets": [
        {
            "name": "debug",
            "displayName": "Debug",
            "configurePreset": "devel",
            "configuration": "Debug",
            "targets": ["ALL_BUILD"]
        },
        {
            "name": "release",
            "displayName": "Release",
            "configurePreset": "default",
            "configuration": "Release",
            "targets": ["ALL_BUILD"]
        }
    ],
    "testPresets": [
        {
            "name": "default",
            "displayName": "Default",
            "configurePreset": "debug",
            "configuration": "Debug"
        }
    ],
    "packagePresets": [
        {
            "name": "default",
            "displayName": "Default",
            "configurePreset": "release",
            "configurations": ["Release"],
            "generators": ["ZIP"],
            "packageDirectory": "${sourceDir}/artifacts"
        }
    ],
    "workflowPresets": [
        {
            "name": "ci",
            "displayName": "Continuous Integration",
            "steps": [
                { "type": "configure", "name": "devel" },
                { "type": "build", "name": "debug" },
                { "type": "test", "name": "default" }
            ]
        },
        {
            "name": "package",
            "displayName": "Package Release",
            "steps": [
                { "type": "configure", "name": "release" },
                { "type": "build", "name": "release" },
                { "type": "package", "name": "default" }
            ]
        }
    ]
}
```

There are a few conventions I've found useful when defining presets.



- The first is using a hidden base configure preset to define the common
shared options like build and install directories or options to be passed
to vcpkg.



- The next is placing the build, install, and vcpkg directories into a
artifacts folder. This is essentially the same as the `out/` folder which is
used by default in visual studio and simplifies `.gitignore` or similar files.

- And the last is naming the build and install directories after the current 
preset so it's possible to build and test different variants without 
overwriting anything.



Using this setup provides as close to a zero-configuration environment
for new developers as I've presently been able to achieve and it also
simplifies CI/CD stages into single `cmake --workflow --preset=` command.

While this is a massive improvement over what was possible even five years ago
there are still a few areas where there's room for improvement.



CMake 3.24 introduced support for dependency providers. They allow for
intercepting calls to find_package through an api. I'd like to see if it would
be possible to develop a system based on dependency providers that uses the 
find_package definitions as the source of dependency information instead of 
an external manifest file.



I'd also like to see a tool for managing compilers and build tools in a
declarative fashion. It appears there has been some work done on this front
with vcpkg artifacts but it seems like development has stalled with only
a small number of embedded environments supported.



The default behaviour of vcpkg is for all dependencies to be built as either
a shared or static library. Finer grained control of this is possible by 
writing a custom toolchain file that specifies the linkage type for specific 
libraries but there should be an easier way to manage this. For example, I generally prefer to use statically linked libraries since it 
allows for link-time optimization and keeps the install manifest tidy.
However, if I'm writing a plugin-based or multi binary application it's good
to have the common functionality in a shared library.
Alternatively, if I'm relying on a lgpl licensed dependency I could be required
to link to it dynamically to remain license compliant. In either situation 
I still want to gain the benefits of static libraries where possible so 
building these libraries to be linked statically with their transient 
dependencies is preferable. 



To conclude, using vcpkg with cmake presets makes it possible to manage your 
dependencies and configure your development environment in a declarative
fashion. There are still a few weak areas but it's possible to work around most
of the pain points in a clear fashion. Overall, the ecosystem is in a much 
better place than it was even a few years ago and I'm looking forward to see
it's continued evolution.