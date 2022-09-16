---
title: Simplify your C++ development environment using CMakePresets.json
date: 2022-09-16T13:49:10.019Z
draft: true
longform: true
tags:
  - c++
  - cmake
---


CMake is a cross-platform utility that abstracts your C++ projects across a wide array of operating systems, compiler toolchains, and build tools. While supported by popular editors such as Visual Studio, VSCode, and CLion, there hasn’t been any way to specify build variants and toolchain options in a cross-editor fashion.

Build variants is the  collection of the all the different ways your project can be built. This includes Debug/Release builds, builds with various compile-time options enabled or disabled, and builds with different compiler toolchains, whether that be for Windows, Mac, Linux, Mobile, or Embedded platforms.

Normally each variant needs to be configured using a set of options passed to CMake during the configuration step, or through an editor specific means of configuration such as cmake-variants.json for VSCode or CMake profiles in CLion. Having these different formats adds an additional point where consistency has to be ensured in a code base to make sure that for example, a debug build on Linux using GCC is compiled with the same options as a debug build on windows using MSVC.

Starting with CMake 3.19 (released November 2020) a new feature called CMake Presets has been added to describe CMake configuration options in a universal way. This makes it easier to maintain consistent build configurations across different developer setups and CI/CD pipelines.

You can use CMake presets by creating a file named CMakePresets.json in your project root directory. The basic format for this file is:

```json
{
  “version”: 2
  “cmakeMinimumRequired”: {
    “major”: 3,
    “minor”: 20,
    “patch”: 0
  },
  “configurePresets”: [
    ...
  ]
}
```

From my experience testing, using version 2 of the schema has the best compatibility across multiple editors as of this writing (September 2022).

To define a preset within the *configurePresets* array we create a json object with the following format

```json
{
  “name”: “preset-name”,
  “displayName”: “Pretty name for editor GUI”,
  “description”: “Tooltip for editor”,
  “generator”: “Name of build system to generate for, argument to -G flag of cmake command”,
  “binaryDir”: “build directory, can use ${sourceDir} to specify it as relative to project root directory”,
  “cacheVariables”: {
     “CACHE_VAR1”: “equivalent to -DCACHE_VAR1=”,
     “CACHE_VAR2”: {
        “type”: “STRING”,
        “value”: “equivalent to -DCACHE_VAR2:STRING=”
     }
  },
  “environment”: {
     “ENV_VAR1”: “same format as the cache variable entries”
  }
}  
```

The variables for a preset map directly to the arguments that would have previously been passed to the cmake command.

To use a preset from the command line, pass the –prefix=prefix-name argument to the cmake command. Any other arguments passed will override conflicting variables in the preset. As an example:

Before:

```shell
mkdir build && cd build && cmake .. -GNinja -DCMAKE_BUILD_TYPE=RelWithDebInfo -DBUILD_TESTING=ON
```

A﻿fter:

```shell
cmake . --preset=preset-name
```

If you’re using an editor that supports presets, all you need to do is open the project and it will handle everything for you.

Now you’ve seen how to create a CMakePresets.json file and define basic presets. But what about for projects with more complicated feature matrices?

With the basic pattern presented before, you would need to define one preset for each combination of your build options, which introduces the need to ensure consistency for the feature options across every preset with it enabled. It would be nice to encapsulate the options specific to each feature in a reusable manner which we can do using preset inheritance.

As an example, if we have a feature matrix along two axis of build type, and c++ compiler we end up with N + M feature presets followed by one preset for each allowed state of the feature matrix.

```json
{
    "version": 2,
    "cmakeMinimumRequired": {
        "major": 3,
        "minor": 20,
        "patch": 0
    },
    "configurePresets": [
        {
            "name": "core",
            "hidden": true,
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/out/build/${presetName}",
            "installDir": "${sourceDir}/out/install/${presetName}"
        },
        {
            "name": "debug",
            "hidden": true,
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug"
            }
        },
        {
            "name": "release",
            "hidden": true,
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release"
            }
        },
        {
            "name": "msvc",
            "hidden": true,
            "cacheVariables": {
                "CMAKE_CXX_COMPILER": "cl.exe"
            }
        },
        {
            "name": "gcc",
            "hidden": true,
            "cacheVariables": {
                "CMAKE_CXX_COMPILER": "g++"
            }
        },
        {
            "name": "debug-msvc",
            "displayName": "Debug (MSVC)",
            "inherits": [
                "core",
                "debug",
                "msvc"
            ]
        },
        {
            "name": "debug-gcc",
            "displayName": "Debug (GCC)",
            "inherits": [
                "core",
                "debug",
                "gcc"
            ]
        },
        {
            "name": "release-msvc",
            "displayName": "Debug (MSVC)",
            "inherits": [
                "core",
                "release",
                "msvc"
            ]
        },
        {
            "name": "release-gcc",
            "displayName": "Debug (GCC)",
            "inherits": [
                "core",
                "release",
                "gcc"
            ]
        }
    ]
}
```

Using this strategy for presets makes it easier to incorporate into CI/CD matrix builds since the preset name can be templated to have a command like:

```shell
PRESET="${feature-axis-1}-${feature-axis-2}"
cmake . --preset=$PRESET && cmake --build --preset=$PRESET
```

Presets are a relatively new feature and their capabilities are still evolving so in time there may be better approaches for handling large feature matrices, but for now you can still use presets to simplify your cross platform project configuration needs and get one step closer to having robust, parametric builds for your software projects.

For a full specification of the CMakePresets.json format and allowed properties check the official documentation here: <https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html>