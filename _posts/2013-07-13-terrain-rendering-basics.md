---
layout: post
category: graphics
tags: [terrain, GLSL]
---
In the previous post about the <a href="{% post_url 2013-08-18-terrain-rendering-with-GLSL %}">heightmap concept</a>  you saw how to create a C++ class template for a heightmap.  This post is the next step: you'll gain an understanding of how to use OpenGL to render a terrain that uses that heightmap.


<img src="http://lh3.ggpht.com/-2oU6PugYIbU/UeHGZOVlRVI/AAAAAAAAAPY/DHO9xl1NJZc/screen_thumb.png?imgmax=800" style="float:right"/>
The screenshot on the right shows what you'll have if you follow this post carefully.  The screenshot is produced from the heightmap concept and the BMP file introduced in the previous post.

Broadly speaking, the terrain is rendered in two steps: 1) create the geometry 2) send the geometry to OpenGL.   The first step in technology agnostic and and could be of use even if you do not like OpenGL.   

## Geometry
Let's start with step one. Terrain geometry needs vertices, so you need to use each element in the heightmap matrix to create a three-dimensional vertex.  This is not difficult because a matrix element can be represented as the triple: \\((c,r,h)\\) where \\(c\\) is the column, \\(r\\) is the row and $h$ is the value of the element.  Imagine \\(h\\) is zero for all matrix elements, then the terrain is a flat rectangular grid, composed of equally sized squares. If there are \\(m\\) columns and \\(n\\) rows in the heightmap there are \\(m \times n\\) vertices, and consequently you'll have \\((m-1) \times (n-1)\\) squares in your terrain.  You can easily see how it works if the heights are not zero imagining that the corners of the squares are pulled up based on the value of h.  Clearly your imagination with not be sufficient in itself: you need a function that maps each triple \\((c,r,h)\\) to a vertex \\((x,y,z)\\). I call this function the transformer.

Before you can define the transformer, you need to commit to a meaning for the components of the vertex \\((x,y,z)\\).  When talking about terrains, I find it useful to think in terms of the cardinal directions.   Orientate your terrain so that the top row of squares lies north and the left column of squares is on the west side.  OpenGL does not know the meaning of the cardinal directions, so we need to decide on a Cartesian coordinate system that maps nicely to these directions.  Let \\(x\\) increase from east to west, \\(y\\) increase from south to north and \\(z\\) increase upwards.  Take a moment to consider what this means. You should also decide on an origin for the vertex on the north-west corner: let that be \\((0,0,h')\\) where \\(h'\\) is a mapping from the \\(h\\) value of the matrix element at \\((0,0)\\).

The transformer needs to know the size of your terrain.  OpenGL does not define units, but it is a good idea for you to tie meaning to the floating point numbers sent to the rendering engine.  For convenience I always use the rule: one meter = `1.0f`.  Let `square_length` be the length of the side of a square.  So, if your terrain stretches 10km west to east and you have 20 squares in a row, then your `square_length` is `500.0f`.   Using this value, the transformer can calculate two components: `x = r * square_length` and `y = c * square_length * (-1)`.  Notice that the sign for `y` is changed because `r` increases towards the south while the coordinate system's `y`increases towards the north.

The mapping from \\(h\\) to \\(z\\) should also be uncomplicated.  If you use a byte for \\(h\\) the value of \\(h\\) ranges from 0 to 255.  One way to use this range is to decide on the maximum and minimum height you want for your terrain. Let's call those `max_h` and `min_h`. Keep in mind that `min_h` could be negative and that a negative value for \\(h\\) could mean below water level.  From these bounds you calculate
`height_scale = (max_h / min_h) / 255`, and you get the function `z = h * height_scale` which calculates the final component of the vertex.   

Listing 1 shows the functor template called terrain::Transformer.  This class implements the component calculations in C++.  The subclass called terrain::TransformerByte can be used for a heightmap with byte elements. It simply provides a convenient constructor that takes the bounds of h as arguments.  The basic Byte, Scalar and Vector types used in the listing are defined in GameEx.

**Listing 1:**
{% highlight cpp %}
template <typename elemT> class Transformer {
public:
	Transformer(const Scalar& square_length, const Scalar& height_scale)
		: _square_length(square_length), _height_scale(height_scale) {}
	Scalar x(int c) const { return c * _square_length; }
	Scalar y(int r) const { return r * _square_length * (-1); }
	Scalar z(int h) const { return h * _height_scale; }
	Vector operator()(int c, int r, elemT h) const {
		return Vector(x(c),y(r),z(h));
	}
	virtual ~Transformer() {}
private:
	const Scalar _square_length;
	const Scalar _height_scale;
};

class TransformerByte : public Transformer<game::Byte> {
public:
	TransformerByte(const Scalar& square_length,
			const Scalar& height_min, const Scalar& height_max)
	:Transformer<game::Byte>(square_length, (height_max - height_min)/255){}
};
{% endhighlight %}

## OpenGL Triangle Strips
That wraps up step one: you have geometry.  Now we decide how render the geometry.  Let’s create a triangle strip for each column of squares.  Consider the squares in the west-most column. The vertices on left (west) side of those squares have \\(c = 0\\), and those on the right hand side \\(c = 1\\).  For this column you create the triangle strip by walking the following  \\((c,r)\\) sequence: \\((1,0), (0,0), (1,1), (0,1), (1,2), (0.2), (1,3) …. (1,m),(0.m)\\).  
This traversal through the elements in a heightmap, can be implemented on the terrain::Heightmap class template.  Listing 2 shows the new traverse_triangles method.  This method takes a function as argument.  It calls this function for each triple \\((c,r,h)\\) it visits while walking in the desired sequence column by column.  It emits a sentinel triple \\((-1,-1,0)\\) to indicate the end of a column has been reached.

**Listing 2:**
{% highlight cpp %}
void traverse_triangles(std::function<void (int c, int r, elemT h)> applyF) const {
	for (auto c = 0U; c < m_columns-1; c++) {
		auto nc = c + 1;
		for (auto r = 0U; r < n_rows; r++) {
			applyF(nc,r,_elems.at(r * n_rows + nc));
			applyF(c,r,_elems.at(r * n_rows + c));
		}
		applyF(-1,-1,0); // signal end of triangle strip
	}
}
{% endhighlight %}

Listing 3 combines the geometry and rendering approach in a class template called TerrainObject.  Take a close look at the draw method: it first draws solid triangles and then red lines ‘over’ those triangles.  Notice how traverse_triangles is called in line 19: it uses a [C++ lambda function](http://www.cprogramming.com/c++11/c++11-lambda-closures.html).

**Listing 3:**
{% highlight cpp linenos %}
template <typename elemT, typename heightmapT, typename transformerT> class TerrainObject
  	: public GameObject {
public:
	TerrainObject(heightmapT * hmap, const transformerT& transformer)
		: _hmap(hmap), _transformer(transformer) {}
	void draw(const DrawContext& gc) override {
		glPolygonMode(GL_FRONT, GL_FILL);
		glColor3b(100,100,100);
		draw_strips();
		glPolygonMode(GL_FRONT, GL_LINE);
		glEnable(GL_LINE_SMOOTH);
		glLineWidth(2);
		glColor3b(50,0,0);
		draw_strips();
	}
protected:
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
	std::unique_ptr<heightmapT> _hmap;
private:
	transformerT _transformer;
};

{% endhighlight %}

Listing 4 shows how these concepts are brought together in the GameEx framework.  If you use another game library this part would obviously be vastly different.  The lines in the listing is all that is needed to show the terrain and have a crude camera to explore your creation a bit.

**Listing 4:**
{% highlight cpp %}
typedef HeightmapWithByte<50,50> demo_hm_t;

class DemoTerrain : public TerrainObject<Byte,demo_hm_t,TransformerByte> {
public:
  DemoTerrain() : TerrainObject(
			new demo_hm_t(),
			TransformerByte(1.0f,0.0f,5.0f))  {}
	void initialise(const ResourceContext & rctx, const DrawContext& dctx) override {
		_hmap->read_from_bmp(rctx.dir() + "test.bmp");
	}
};

class TerrainController : public MainObject {
	public:
	TerrainController() : MainObject(-100,0.3,100) {
		add_part(new SphereCamera(-1,10.0f,50.0f, Vector(25.0f,0.0f,0.0f)));
		add_part(new DemoTerrain());
	}
};

int main( int , char* [] ) {
	game::Game g(new TerrainController(), "terrain/", 50, 5000);
	return g.run();
}
{% endhighlight %}

This concludes the post. Using the BMP file as as input,  you created a three-dimensional image that has 4900 triangles.  And maybe you leant a bit more of the new C++ language features.
