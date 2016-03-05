---
layout: post
category: graphics
tags: [OpenGL, terrain, GLSL]
---
In a previous <a href="{% post_url 2013-07-29-faster-terrain-rendering %}">post about faster terrain rendering</a> I used [`glDrawArrays`](http://www.opengl.org/wiki/GLAPI/glDrawArrays) for rendering each column.  Here we talk about a faster way.


To work with `glDrawArrays`, we had to put each vertex we drew into a buffer. The thing is that each vertex is repeated in this array every time it is drawn.  For example, if we have  \\(128 \times 128\\) heightmap, we have \\(16384\\) vertices.  Each of the \\(128\\) columns (minus one), adds \\(128 \times 2\\) vertices to the buffer.  This is a buffer size of \\(127 \times 128 \times 2 = 32512\\).  Each vertex eats \\(16\\) bytes, coming to an additional payload of about \\(254\\) kb on the GPU.  Things could be much worse if we store more than the location (such as the colour and normals) for each vertex.

There is a simple optimisation to save some space: store the vertices in a an _array buffer_ on the GPU and then draw using indexes to this buffer.  And if you store the index sequence also in a buffer which we'll call the _element buffer_, then the performance impact is small.  This process of drawing from these two buffers is called _index drawing_.   Note that we still use up memory for \\(16384\\) vertices in the _array buffer_.  But, the _element buffer_ now carries the drawing payload, and this could be elements of integers.  That is \\(130048\\) bytes.  Boils down to an additional \\(130\\) kb payload instead of the original \\(254\\) kb.

For OpenGL this means you use a `GL_ARRAY_BUFFER` to store the _array buffer_ and a `GL_ELEMENT_ARRAY_BUFFER` to store the _element buffer_. These two buffers are associated to a bound _vertex array_ (the last invocation of [`glBindVertexArray`](http://www.opengl.org/wiki/GLAPI/glBindVertexArray)) via calls to [`glBindBuffer`](http://www.opengl.org/wiki/GLAPI/glBindBuffer).  During intialisation, you copy data to the buffers, and during the render step, you bind the vertex array again and call [`glDrawElements`](http://www.opengl.org/wiki/GLAPI/glDrawElements) to do the _index drawing_ for you.  These ideas are illustrated in pseudo listing 1.

Notice the _vertex attribute_ calls that involves the value called `position`.  This variable is a reference to a declaration in the GLSL shader (the code is shown in listing 2 below).  Its value is obtained from a call like this `position = glGetAttribLocation(program,"position")`

**Listing 1:**
{% highlight cpp %}
// setup code
dc.gl().glGenVertexArrays(1, &array);
dc.gl().glBindVertexArray(_array);

dc.gl().glGenBuffers(1,&array_buf);
dc.gl().glBindBuffer(GL_ARRAY_BUFFER, array_buf);  
dc.gl().glBufferData(GL_ARRAY_BUFFER,array_size*sizeof(GLfloat), array_data,GL_STATIC_DRAW);
dc.gl().glEnableVertexAttribArray(position);
dc.gl().glVertexAttribPointer(position, 3, GL_FLOAT, GL_FALSE, 0, 0);

dc.gl().glGenBuffers(1,&index_buf);
dc.gl().glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, index_buf);  
dc.gl().glBufferData(GL_ELEMENT_ARRAY_BUFFER,element_size*sizeof(GLuint), element_data,GL_STATIC_DRAW);

// draw code
gl.glBindVertexArray(array);
const auto prims = rows * 2 * sizeof(GLuint);
for (size_t i = 0; i < cols-1;i++)
	glDrawElements(GL_TRIANGLE_STRIP,rows*2,GL_UNSIGNED_INT,(void*)(i*prims));
{% endhighlight %}

**Listing 2:**

	// Vertex shader
	#version 140

	in vec3 position;

	varying float height;
	void main()
	{
		vec4 vertex = vec4(position.xyz,1);
		gl_Position = (gl_ModelViewProjectionMatrix * vertex);
		// normalize height component
		height = (position.z + 1)/10;
	}
