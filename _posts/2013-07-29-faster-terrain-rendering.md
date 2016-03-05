---
layout: post
category: graphics
tags: [OpenGL,C++,terrain]
---
In the previous post about the <a href="{% post_url 2013-07-13-terrain-rendering-basics %}">basics of terrain rendering</a> you saw a simple way to render a terrain in OpenGL.   In this post you'll see how to improve the performance of the render procedure.   

On my computer, the original procedure measured 590 frames per second; and with the improvements shown here it measured a whopping 1017 frames per second.   That is nearly a 100% improvement.

Let me start by reviewing the original plan: it is shown in Listing 1.  This procedure traverses through all elements, calculates and sends each vertex to OpenGL.  The CPU does all the calculation work – and it does this every time the scene is rendered.  To be fair, the procedure is not all bad: it uses very little memory.

But what exactly is our memory overhead?  A float is 4 bytes.  As shown in the previous post, a 50x50 heightmap produces 5 000 vertices (during the triangular traverse).  So, if we store each vertex, we would need less than 60 kb (5 000 * 3 * 4) of memory on the GPU.   As a side effect: once we have the vertices on the GPU memory, we do not have to make the 5 000 calls o OpenGL.  And voila, we gain a lot of speed!

**Listing 1:**
{% highlight cpp %}
void draw_strips() {
	glBegin(GL_TRIANGLE_STRIP);
	_hmap->traverse_triangles([&](int c, int r, elemT h) {
		if (c == -1) {
			glEnd();
			glBegin(GL_TRIANGLE_STRIP);
		} else {
			const Vector v = _transformer(c,r,h);
			glVertex3f(v.x(),v.y(),v.z());
		}
	});
	glEnd();
}
{% endhighlight %}

The first task is to calculate the vertices.  This is done once: before this first render.  The idea is simple: store the calculations to a `std::vector`.  The code is shown in Listing 2.  Note that the space for the `vector` is allocated on its constructor – because we know how many elements we need.  

**Listing 2:**
{% highlight cpp %}
const auto b_size = m_columns * n_rows * 2 * 3;
vector<GLfloat> verts(b_size);
int i = 0;
_hmap->traverse_triangles([&](int c, int r, elemT h) {
	if (c != -1) {
		const Vector v = _transformer(c,r,h);
		verts[i++] = v.x();
		verts[i++] = v.y();
		verts[i++] = v.z();
	}
});
{% endhighlight %}

Now that you have the values in main memory, you need to copy the data to the GPU.  This is done in three steps (see Listing 3).  First use [`glGenBuffers`](http://www.opengl.org/wiki/GLAPI/glGenBuffers) to create a handle to a new buffer.  Then, [`glBindBuffer`](http://www.opengl.org/wiki/GLAPI/glBindBuffer) tells the OpenGL state machine that you want to use this buffer.  And then [`glBufferData`](http://www.opengl.org/wiki/GLAPI/glBufferData) does the real work: it copies the data from the main to the GPU memory.  The last line tells OpenGL that we are not using the buffer anymore.    

**Listing 3:**
{% highlight cpp %}
glGenBuffers(1,&_buffer);
glBindBuffer(GL_ARRAY_BUFFER, _buffer);
glBufferData(GL_ARRAY_BUFFER,b_size*sizeof(GLfloat), (void*) verts.data(),GL_STATIC_DRAW);
glBindBuffer(GL_ARRAY_BUFFER, 0);
{% endhighlight %}

All that remains for you is to render the scene using the buffer.  This is done in Listing 4.  This procedure must be called inside the game loop for every render.  As before you tell OpenGL you want to use the buffer identified by the previously allocated handle. The calls to `glEnableVertexAttribArray` and `glVertexAttribPointer` tells OpenGL what the format is of the data in the buffer:  essentially we store 3 `float` values per vertex.  Then the call to `glDrawArrays` uses the data definition and the data in the buffer to render the terrain.  It is called for each column.  You use the heightmap dimensions to calculate prims, the number of vertices per column.  This value is then used to find the offset in the data for the given column.

**Listing 4:**
{% highlight cpp %}
glBindBuffer(GL_ARRAY_BUFFER,_buffer);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
// draw column per column
const auto cols = _hmap->count_columns();
const auto rows = _hmap->count_rows();
const auto prims = rows * 2;
for (size_t i = 0; i < cols;i++)
	glDrawArrays(GL_TRIANGLE_STRIP,i*prims,prims);
glDisableVertexAttribArray(0);
{% endhighlight %}

You now know a little bit more about OpenGL buffers.  You get the same picture as before – its just faster.
