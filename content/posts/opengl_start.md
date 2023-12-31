+++
title = "A comprehensive guide of basic opengl"
date = 2023-10-20T16:35:00-05:00
draft = false
summary = "A simple guide to help you kickstart your opengl development journey"
author = "Katherine Chesterfield"
+++
## Contents

{{% toc %}}

## Inspiration
This is the part where I tell you the story of my life, feel free to skip it, **but** know it will break my heart 💔

joking aside C++ and OpenGL have always been goals that I wanted to achieve in my programming journey, I'd like to thing I've got C++ covered to *some* extent, but truth is, **no one really knows C++**, but that's besides the scope of this article.

I was so enthusiastic to the point of first trying to learn C++ before I was even in high school, needless to say I never got past the basics so it's safe to say I failed back then.

Nowadays it became an easier task to tackle with more years of experience under my belt, that and the ability to seek out solid resources that helped have a deep understanding of the core concepts of the language, but as any C++ developer will tell you, I don't kno anything yet, and probably never will.

There was this youtube series by [Hopson](https://www.youtube.com/@Hopsonn) where they recreated minecraft from scratch using C++ and OpenGL, I was mesmerized by their skills and wanted to make a minecraft clone of my own, at some point I decided to try and follow an OpenGL tutorial (specifically by [The Cherno](https://www.youtube.com/@TheCherno)), failing once again, mostly because of my ignorance regarding the C++ tooling environment.

Coming back years ago with some C++ knowledge and having worked with higher level graphics frameworks, I decided to give it a go once again, now being able to understand I decided to write this article to document this first stage of learning while strenghtening my knowledge by explaining my findings.

## Requirements

While it might not seem this way, the OpenGL experience on windows and Linux isn't all that different, so regardless of your platform this article could be a decent resource to help you get started.

I'll be using Arch Linux alongside neovim, cmake (with make) 3.16, clang 16 and opengl 4.6.

For the actual project I'll use glfw3 for window management, and glad for opengl extension loading.

## Setting up

Sadly setting up an environment for OpenGL development is quite tedious, I'll do my best to make this task as easy as possible.

This first part of the guide will focus on how to set up on linux, for windows users I would recommend following the first 2 videos of [This tutorial](https://www.youtube.com/playlist?list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2) by The Cherno, afterwards you can skip directly to [the pipeline](#the-opengl-pipeline-in-a-nutshell).

### Disclaimer
There is a myriad of ways to set up a C++ development enviroment, today I'll do what's most straightforward but any other way should be fine if you know what you are doing.
### Creating our project
Writing a new `CMakeLists.txt` is the first step of every project, before that though, this is the structure of the project we'll be using.
```
Opengl/                                          
 ├─CMakeLists.txt
 ├─build/
 │  └─...
 ├─libs/
 │  ├─include/
 │  │  ├─GLFW/
 │  │  │  └─...
 │  │  └─GLAD/
 │  │     └─...
 │  └─bin/
 │     ├─libglfw3.a
 │     └─glad.a
 └─src/
    └─Main.cpp
```

we'll get there in the following steps, for now create your `OpenGL` folder, inside create the `build libs src` folders and the `CMakeLists.txt` file

```sh
$ mkdir OpenGL && cd OpenGL
$ mkdir build libs src && echo "" >> CMakeLists.txt && echo "" >> src/Main.cpp
```

now we'll create a simple cmake file, we'll extend it eventually

```cmake
# Start with the minimum version, I recommend 3.16
cmake_minimum_required(VERSION 3.16...3.27)
# Then project metadata
project(ogl LANGUAGES CXX)

# Here I'll get the device architectury, this part is optional but will help us organize the project
SET(ARCHITECTURY 32)

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(ARCHITECTURY 64)
endif()

# Build output configuration
SET(CMAKE_BUILD_TYPE Debug)
SET(FULL_OUTPUT_DIR "${CMAKE_SOURCE_DIR}/build/${CMAKE_SYSTEM_NAME}/${CMAKE_BUILD_TYPE}-${ARCHITECTURY}")
# Alternatively
# SET(FULL_OUTPUT_DIR "${CMAKE_SOURCE_DIR}/build/${CMAKE_SYSTEM_NAME}/${CMAKE_BUILD_TYPE}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${FULL_OUTPUT_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${FULL_OUTPUT_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${FULL_OUTPUT_DIR}/bin")

# If you're using clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_CXX_EXTENSIONS ON)

# change `-stdlib=libc++` for your standard library, the `-I` parameter is for our clangd later on
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -I${CMAKE_SOURCE_DIR}/libs/include")
add_compile_definitions(SOURCE_DIRECTORY=${CMAKE_SOURCE_DIR})

# We'll come back here to find our libraries

# You may be used for globbing your source files, however it's not recommended
# instead define each source file by hand
set(SOURCES src/main.cpp)

# This will be our application
add_executable(ogl ${SOURCES})
target_compile_features(ogl PRIVATE cxx_std_20)
target_include_directories(ogl PUBLIC ${CMAKE_SOURCE_DIR}/libs/include)
```

Feel free to write hello world in our main source file to make sure things work as intended.
```cpp
#include <iostream>

// I prefer trailing return types but feel free to use the more traditional
// int main()
auto main() -> int
{
	std::cout << "Hello World!" << '\n';
}
```

Now, if you're using clangd as your diagnostics LSP, we need to symlink our `compile_commands.json` to our root folder.

Make sure to build cmake beforehand.

```sh
# make sure you're inside your project folder
$ cmake -B build && cmake --build build
$ ln -sf build/compile_commands.json ./compile_commands.json
```

Lastly, run the executable inside `./build/Linux/Debug-64`, remember to replace with your OS and architecture.
```sh
./build/Linux/Debug-64/oglpg
```
Should output.
```
Hello World!
```
### GLFW
GLFW calls itself "an Open Source, multi-platform library for OpenGL, OpenGL ES and Vulkan development on the desktop", for now we'll be using it for our window management, but keep in mind it provides an API for much more, such as mouse and keyboard interaction for instance.

The easiest way to use GLFW in your project is to use a package manager (eg. `apt` or `pacman`) but to provide a more universal experience I'll build it from source.

First you have to install some dependencies required to buid GLFW, for this you need to follow the distro specific instructions in the [GLFW docs](https://www.glfw.org/docs/latest/compile.html#compile_deps_x11), however if you're using arch linux aswell all you need to do is install the `xorg` package (as long as you're running X11 on your system).

The easiest way to use GLFW now is to use it as a cmake submodule, if your root folder is a git repository, run the following command.
```sh
git submodule add https://github.com/glfw/glfw.git ./libs/glfw
```
Otherwise run.
```sh
git clone https://github.com/glfw/glfw.git ./libs/glfw
```

now we have to tell cmake where to find glfw, it's super simple, all we need to do is

```cmake
...
# We'll come back here to find our libraries
add_subdirectory(libs/glfw)
find_library(glfw NAMES glfw glfw3 libglfw libglfw3)
...
```

we'll link with our main target later

### GLAD
GLAD is lovely to setup, it provides a simple CLI interface written in python to generate a custom loader for our needs. 
Installation is pretty straight forward
For arch linux users:
```sh
yay -S python-glad
```
And for everyone else:
```sh
sudo pip3 install glad
```

now we'll generate our files, we'll keep it simple since it provides some *complicated* options that to be honest, I'm unsure how to use.

Create a new folder inside `libs/` called `glad`, there we'll run the command.
```sh
glad --profile core --out-path . --generator c
```

this command generates 2 folders, `include` and `src`, to use this in our project we'll need to create a new `CMakeLists.txt` that declares our `glad` target, it's pretty straight forward so don't stress.

```cmake
cmake_minimum_required(VERSION 3.16...3.27)
project(glad LANGUAGES C)

add_library(glad STATIC src/glad.c)
target_include_directories(glad PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include")
```

Lastly we add the library to our main `CMakeLists.txt`.
```cmake
...
# We'll come back here to find our libraries
add_subdirectory(libs/glfw)
add_subdirectory(libs/glad)
find_library(glfw NAMES glfw glfw3 libglfw libglfw3)
find_library(glad NAMES glad libglad)
...
```
### Wrapping up
To finish up, all we have to do is include the directories in our main executable and link the libraries
```cmake
...
# This will be our application
add_executable(ogl ${SOURCES})
target_compile_features(ogl PRIVATE cxx_std_20)
# Include the new folders
target_include_directories(ogl PUBLIC ${CMAKE_SOURCE_DIR}/libs/include ${CMAKE_SOURCE_DIR}/libs/glad/include ${CMAKE_SOURCE_DIR}/libs/glfw/include)
# Link the libraries
target_link_libraries(ogl PRIVATE glad glfw)
```
Remember that we're adding our source files manually. So don't forget to add them to you `CMakeLists.txt` `SOURCES` variable everytime you add or remove a file.

Now we should be ready to start working on OpenGL.
## Hello OpenGL
Here I'll borrow some code from the great [LearnOpenGL](https://learnopengl.com) to rush through the hello world.
```cpp
// main.cpp
#include <iostream>
#include "GLFW/glfw.h"
#include <glad/glad.h>

auto main() -> int
{
	// First we initialize glfw with the correct version and profile hints
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
	glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

	// Then we create the window and verify its valid
	auto* window = glfwCreateWindow(800, 800, "Hello OpenGL", nullptr, nullptr);
	if (window == nullptr)
	{
		std::cerr << "Failed to initialize GLFW window" << '\n';
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(window);

	if(!gladLoadGLLoader(reinterpret_cast<GLADloadproc>(glfwGetProcAddress)))
	{
		std::cerr << "Failed to initialize GLAD" << '\n';
		return -1;
	}

	// Telling opengl size of the window
	glViewport(0, 0, 800, 800);

	// Main loop, will run as long as the window is open
	while(!glfwWindowShouldClose(window))
	{
		// Swapping the buffers (You'll understand this later)
		glfwSwapBuffers(window);
		glfwPollEvents();
	}
	glfwTerminate();
}
```
We could add these 2 lines of code before swapping the buffers to change the color of the window.
```cpp
// Setting the color to a pink shade
glClearColor(1.0f, 0.8f, 0.8f, 1.0f);
// Resetting the color buffer
glClear(GL_COLOR_BUFFER_BIT);
```
If everything went correctly, a window should open when you build and run.
```sh
$ cmake --build build
$ ./build/main
```
And it should look like this.

![showcase of the window](https://raw.githubusercontent.com/KatieUmbra/sandbox-blog/main/assets/opengl%20window.png)

Congrats on finishing the first part of this guide. However much more needs to be done before we see a triangle (or triangles) in the screen.

That's it for now, part 2 is on the way.
