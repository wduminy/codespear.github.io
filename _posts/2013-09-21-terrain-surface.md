---
layout: post
category: graphics
tags: [terrain]
---
In this post we take a look at how we can determine the location of a point on the surface of a terrain.


Recall from the <a href="{% post_url 2013-07-13-terrain-rendering-basics %}">terrain basics post</a> that we decided on a coordinate system where \\(z\\) points upwards, \\(x\\) points west and \\(y\\) points south.  So using this coordinate system we need to determine the \\(z\\) value, given the \\((x,y)\\) world coordinate.
<img src="/assets/images/2013/terrain-surface.jpg" style="float:right"/>

It helps to think of this problem as a two step process. First, you need to identify the triangle in the heightmap for the coordinate \\((x,y)\\).  Then you find the point of intersection between the triangle's plane and the line that drops down vertically through \\((x,y)\\).   

Using the ideas described here, I placed some vertical poles in my terrain.  The resulting image is shown here.

## Step 1: Identify the triangle
The way you identify the triangle depends on the way you drew them.  Assuming you used the rendering algorithm I described in the <a href="{% post_url 2013-07-13-terrain-rendering-basics %}">mentioned post</a>, you can start with world coordinates \\((x,y)\\) and translate it to \\((c,r)\\) so that the latter is the heightmap coordinates.  For example, if you have \\(10\\) world units per heightmap unit, your world coordinates \\((23,24)\\) would translate to the heightmap coordinates \\((2.3,2.4)\\).  Take the [`floor`](http://www.cplusplus.com/reference/cmath/floor/) of these \\((c,r)\\) coordinates and you end up with the answer that the triangle is in column 3, row 3 of the matrix.  That is in zero based index \\((2,2)\\).  There are two triangles for that matrix element; we need to decide which one contains \\((c,r)\\).  This answer is illustrated in Listing 1.  Note that there is a corner case handled at the start of the listing (for when \\((r,c)\\) is in the corner :-) ).  

**Listing 1:**
{% highlight cpp %}
Triangle find(float c, float r) {
	if (r == n_rows - 1 || c == m_columns - 1) {
		new_c = (c == m_columns-1)?(c - 0.001f):c;
		new_r = (r == n_rows-1)?(r - 0.001f):r;
		return find(new_c,new_r,applyF);
	} else {
		const int fc = (int) floor(c);
		const int fr = (int) floor(r);
		const Scalar x = c - fc;
		const Scalar y = r - fr;
		if (y <= x)
			return new Triangle(point(fc,fr),point(fc+1,fr),
				point(fc+1,fr+1));
		} else
			return new Triangle(point(fc,fr+1),point(fc,fr),
				point(fc+1,fr+1));
		}
	}
}

Vector point(int c, int r) {
	return new Vector(c,r,height_at(c,r));
}
{% endhighlight %}

## Step 2: Point of intersection
When the opportunity arises, it is always a good idea to review our understanding of geometry a bit. Let the identified triangle be the vectors \\(a\\), \\(b\\) and \\(c\\). The normal for the plane that contains these vectors is \\(n = (a - b) \times (c - b)\\).  An equation for the plane is then \\((v - b) \cdot n = 0\\).  Let \\(v = (x,y,z)\\), then the solution for \\(z\\) in the plane equation is this rather long expression:

\\(\frac{( ( b_x-a_x) c_z+( a_z-b_z) c_x+a_xb_z-a_zb_x) y+( ( a_y-b_y) c_z+( b_z-a_z) c_y-a_yb_z+a_zb_y) x+( a_xb_y-a_yb_x) c_z+( a_zb_x-a_xb_z) c_y+( a_yb_z-a_zb_y) c_x}{( b_x-a_x) c_y+( a_y-b_y) c_x+a_xb_y-a_yb_x}\\)  

Note that \\(x\\) and \\(y\\) is rather sparse in this expression.  We can simplify as follows

\\( z = (e_1 y + e_2 x + e_3) \div e_4 \\)

Where

\\(e_1 = ( ( b_x-a_x) c_z+( a_z-b_z) c_x+a_xb_z-a_zb_x)\\)
\\(e_2 = ( ( a_y-b_y) c_z+( b_z-a_z) c_y-a_yb_z+a_zb_y)\\)
\\(e_3 = ( a_xb_y-a_yb_x) c_z+( a_zb_x-a_xb_z) c_y+( a_yb_z-a_zb_y) c_x\\)
\\(e_4 = ( b_x-a_x) c_y+( a_y-b_y) c_x+a_xb_y-a_yb_x\\)

Listing 2 illustrates how this calculation can be implemented. If you code this, it is probably a good idea to check \\(e_4\\) for \\(0\\) before dividing.  If this value is zero, the input parameters do not form a triangle.

**Listing 2:**
{% highlight cpp %}
Vector plane_point(Vector a, Vector b, Vector c, Scalar x, Scalar y) {
	const auto e1 = (( b.x()-a.x()) * c.z()+( a.z()-b.z()) * c.x()+a.x() * b.z()-a.z() *
		b.x());
	const auto e2 = (( a.y()-b.y()) * c.z()+( b.z()-a.z()) * c.y()-a.y() * b.z()+a.z() *
		b.y());
	const auto e3 = (a.x() * b.y()-a.y() * b.x()) * c.z()+( a.z() * b.x()-a.x()
		* b.z()) * c.y()+( a.y() * b.z()-a.z() * b.y()) * c.x();
	const auto e4 = (b.x()-a.x()) * c.y()+( a.y()-b.y()) * c.x()+a.x() * b.y()-a.y() *
		b.x();
	const auto z = (e1 * y + e2 * x +e3)/e4;
	return Vector(x,y,z);
}
{% endhighlight %}

## Putting it together
You should be able to put together the ideas in this post and render something on your terrain. Listing 3 shows how I drew the image shown above.  The listing shows the `draw` method of a class derived from [`terrain::TerrainObject`](/gameex/html/classterrain_1_1_terrain_object.html).  This code is just for demonstration - it would not be a good idea to do all these calculations on the `draw` method.

**Listing 3:**
{% highlight cpp %}
void draw(const game::DrawContext& dc) override {
		base_terrain_t::draw(dc);
	glLineWidth(10.0f); // in pixels
	auto sw = floor_south_west();
	auto ne = floor_north_east();
	const int steps = 200;
	auto inc = (sw - ne) / (steps * 1.0f);
	glBegin(GL_LINES);
	for (int i=0;i<steps;i++) {
		auto vi = ne + (inc*i);
		auto fi = floor_at(vi.x(),vi.y());
		glColor3f(0,0,1); // blue
		glVertex3f(fi.x(),fi.y(),fi.z());  
		glColor3f(1,0,0); // red
		glVertex3f(fi.x(),fi.y(),fi.z()+1);    	
	}
	glEnd();
}
{% endhighlight %}
