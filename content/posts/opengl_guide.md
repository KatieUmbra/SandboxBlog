+++
title = "An OpenGL (Likely OK) Guide"
date = 2023-10-19T11:00:00-05:00
draft = false
summary = "A (likely okay) quick start opengl guide."
author = "Katherine C."
+++

## Vertex Buffers
use the opengl vertex buffer slot to store vertex data such as
positions, color, normals, texture, etc...

first we generate a new buffer with an unique id and give it
some memory to store the buffer identifier

we call this buffer a Vertex Buffer Object. [reference](https://docs.gl/gl4/glGenBuffers)

```cpp
unsigned int vbo;
glGenBuffers(1, &vbo);
```

then we use opengl as a state machine and we select (bind) the
buffer, giving it the type of buffer, eg array. [reference](https://docs.gl/gl4/glBindBuffer)

```cpp
glBindBuffer(GL_ARRAY_BUFFER, vbo);
```

### data
buffers are made to move data to the gpu, so first we need to declare some data.

Here I'll declare a position and uv for each element.

```cpp
std::array<float, 12> vertex_data {
/*      Vertex        UV        */
	-0.5f, -0.5f, 0.0f, 1.0f,
	 0.0f,  0.5f, 0.5f, 0.0f,
	 0.5f, -0.5f, 1.0f, 1.0f
};
```

and then we set the buffer data. [reference](https://docs.gl/gl4/glBufferData)

```cpp
glBufferData(
	// Data Access Hint
	GL_ARRAY_BUFFER, 
	// Size in bytes of the data
	vertex_data.size() * sizeof(float),
	// Pointer to the data
	&vertex_data,
	// Usage Pattern of type GLenum
	GL_STATIC_DRAW
	);
```

## Vertex Attributes

Now we have to define a vertex layout for our buffer

we have 2 attributes in our buffer, a position `vec2` and
an uv `vec2`

```cpp
// position attribute
glVertexAttribPointer(
	// Index of the attribute
	0,
	// Amount of data (eg. 2 floats)
	2,
	// Type of data (GLenum)
	GL_FLOAT,
	// Wether or not we're normalizing to a range
	GL_FALSE,
	// Byte offset: the size of the vertex
	4 * sizeof(float),
	// Pointer INSIDE the attribute
	0
);

// uv attribute
glVertexAttribPointer(
	1,
	2,
	GL_FLOAT,
	GL_FALSE,
	4 * sizeof(float),
	// there are 2 floats before the uv
	static_cast<void*>(2 * sizeof(float))
);
```

then we enable (bind) the attributes
```cpp
glEnableVertexAttribArray(0);
glEnableVertexAttribArray(1);
```

## Shaders

Now we have to write a `frag` shader, and a `vert` shader.

In this case I'll write `glsl` since it's the opengl shader language

**fragment shaders** are "pixel" shaders, these get ran for every
pixel, we use them for color, texture, lighting, etc...

**vertex shader** gets ran for every vertex we send in our buffer.

here I'll just pretend the source code is a `C++` string, ideally
you'd write some code to read from a file and then compile that

we'll define a function that returns an `unsigned int` which is
going to be an opengl program id, said program will compile and
link our shaders

```cpp
static unsigned int create_shader (
	const std::string& vertex_shader,
	const std::string& fragment_shader
	)
{
	auto program = glCreateProgram();
	...
}
```

for simplicity the compilation step will also be abstracted, we'll retake
this function later.

```cpp
static unsigned int compile_shader(GLenum type, const std::string& source)
{
	auto id = glCreateShader(type);
	auto* src = source.c_str();
	glShaderSource(
		// ID of the shader
		id,
		// Amount of shader we're creating
		1,
		// pointer to the source code
		&src,
		// size of the source if not null terminated
		nullptr
		);
	glCompileShader(id);
	// TODO: Error handling
	return id;
}
```

at this step we're not really handling errors, we'll handle that later.

let's go back to our `create_shader`

```cpp
	...
	auto program = glCreateProgram();
	auto vs = compile_shader(GL_VERTEX_SHADER, vertex_shader);
	auto fs = compile_shader(GL_FRAGMENT_SHADER, fragment_shader);

	glAttachShader(program, vs);
	glAttachShader(program, fs);

	glLinkProgram(program);
	glValidateProgram(program);

	// Delete residuals
	glDeleteShader(vs);
	glDeleteShader(fs);

	return program;
```

now error handling. We'll go back to `compile Shader`

```cpp
	...
	glCompileShader(id);
	
	int status;
	// get a shader property
	glGetShaderiv(id, GL_COMPILE_STATUS, &status);
	if (status == GL_FALSE)
	{
		int length;
		glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);
		char* message = static_cast<char*>(
			// Usage of C function to allocate on the stack
			alloca(length * sizeof(char))
		);
		// Logging
		std::cerr 
			<< "[OpenGL](ERROR): Failed to compile "
			// temporary ternary operation
			<< (type == GL_VERTEX_SHADER : "shader" : "vertex")
			<< " shader."
		std::cerr << "[OpenGL](ERROR): " << message << '\n';
		glDeleteShader(id);
		return 0;
	}

	return id;
}
```

let's now create the shaders in our main function, after the vertex attributes.

I'll use a raw string and separate the code snippets.
```cpp
std::string vertex_shader = R"(
```
```glsl
version 460 core

layout(location = 0) in vec2 position;
// at the moment there's no use for the uv
layout(location = 1) in vec2 uvs;

void main()
{
	gl_Position = vec4(position.xy, 0.0, 1.0);
}
```
```cpp
)"

std::string fragment_shader = R"(
```
```glsl
#version 460 core

layout(location = 0) out vec4 color;

void main()
{
	// some RGBA color
	color = vec4(1.0, 0.8, 0.8, 1.0);
}
```
```cpp
)"

auto shader = create_shader(vertex_shader, fragment_shader);
// remember "create_shader" returns a program id
glUseProgram(shader);
```

now we can issue a draw call and
delete our program after we finish our main loop
```c++
while (!glfwWindowShouldClose(window))
{
	...
	glDrawArrays(GL_TRIANGLES, 0, vertex_data.size());
	...
}

glDeleteProgram(program);
```

## Index buffers

while rendering shapes, sometimes 2 shapes share vertices, or
uvs, or positions, etc...

copying this data every time is wasteful, so it's better to use
an index system where we specify the index of the data rather than
the data itself

![index buffer](static/index_buffers.png)

first I'll fix the buffer to contain the vertices of a square

```cpp
std::array<float, 16> vertex_data {
/*      Vertex        UV        */
	-0.5f, -0.5f, 0.0f, 1.0f,
	 0.5f, -0.5f, 1.0f, 1.0f,
	 0.5f,  0.5f, 1.0f, 0.0f,
	-0.5f,  0.5f, 0.0f, 0.0f
};
```

now we need the index buffer data

```cpp
std::array<unsigned int, 6> indices_data {
	0, 1, 2,
	2, 3, 0
}
```

now we create our opengl buffer

```cpp
unsigned it ibo; // index buffer object
glGenBuffers(1, &ibo);
// we're binding to a different "slot"
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);
// storing the data
glBufferData(
	GL_ELEMENT_ARRAY_BUFFER, 
	sizeof(unsigned int) * indices_data.size(),
	indices,
	0
	);
```

now we change our draw call

```cpp
while (!glfwWindowShouldClose(window))
{
	...
	// glDrawArrays(GL_TRIANGLES, 0, vertex_data.size());
	glDrawElements(
		// type of shape
		GL_TRIANGLES, 
		// indice count
		6,
		// Type of data in the index buffer
		GL_UNSIGNED_INT,
		// pointer to the index buffer, since it was bound
		// the argument is not necessary
		nullptr
		);
	...
}
```

## Uniforms

uniforms are just draw call specific data passed from the cpu to a shader

let's try by adding an uniform to our fragment shader

```glsl
#version 460 core

layout(location = 0) out vec4 color;

uniform vec4 uColor;

void main()
{
	// some RGBA color
	color = uColor;
}
```

here im adding a ` vec4 uColor` uniform that takes data from the draw call
and then it sets the output color to that value

now we can send or bind the data, in this case I'll do it before
the main window loop

```c++
...

// uniform unique id
auto location = glGetUniformLocation(shader, "uColor");
// if location is -1 then it's not a valid uniform or if its unused
glUniform4f(
	// uniform uid
	location,
	// vec4 data
	1.0f,
	0.8f,
	0.8f,
	1.0f
	);

while (!glfwWindowShouldClose(window))
{
	...
}
```

## Vertex Arrays

vertex arrays is an opengl exclusive feature, it's rather simple, all it does
is remember some vertex attribute layout to be recalled at any given moment

in recent versions of opengl there must be a vertex array object (vao),
if you use `core` opengl then there must be a `vao` present
otherwise it's not necessary

so now you should enable `core` opengl.

there are two approaches