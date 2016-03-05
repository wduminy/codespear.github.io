---
layout: post
category: graphics
tags: [C++,terrain]
---
If you want to render a terrain, a [heightmap](http://en.wikipedia.org/wiki/Heightmap) is a handy tool to have at your disposal.  This post explores one way to create an abstraction of this concept using the new C++ standard.     

<img src="http://upload.wikimedia.org/wikipedia/commons/5/57/Heightmap.png" style="float:right"/>

In its essence, a heightmap is simply a matrix of numbers where an element at *{c,r}* indicates the height of the terrain at some location.  It is a handy data structure and is also a compact way to store the details of a terrain.   The image on the right hand side is an example of a heightmap shown as a 2D image (taken from Wikipedia). The height is determined by the value of the white color: a whiter pixel indicates a higher level for the terrain at that point.  You should be able to imagine where the hills and valleys are by looking at this image long enough.

As you can expect, compact storage comes with a cost: less data equals less detail.  The granularity of the terrain is determined by the size of the matrix, and also by the size of the data type used for its elements.  For instance if the elements are stored as a bytes the terrain height ranges from 0 to 255.  The height cannot be 23.5 … there is only 256 levels to choose from.  

Getting back to the abstraction: It seems reasonable that a heightmap needs at least three parameters.  These are: (a) the number of columns m, (b) the number of rows n and (c) the data type of the elements, elemT.   We should now be able to implement this abstract concept as a  C++ class template.  

First, we need is to choose a data structure for the matrix data.  We want a structure that is fixed in memory and stores consecutive elements in sequence.  This constraint will make the heightmap very useful for rendering in OpenGL.   The `std::array` container is well suited for this purpose.  

Often we want to address the elements in the array using the {c,r} pair as index.  For this reason, I provided an overloaded () operator that can throw `std::out_of_range`. The code for our class template `Heightmap`, is shown in Listing 1.

**Listing 1:**
{% highlight cpp %}
template <typename elemT, size_t m_columns, size_t n_rows >
class Heightmap {
public:
	typedef std::array<elemT, n_rows * m_columns> container_t;
	const container_t& elems() const {return _elems;}
	container_t& elems() {return _elems;}
	const elemT& operator() (size_t c, size_t r) const {
		check_range(c,r); return _elems.at(r * n_rows + c);}
	elemT& operator() (size_t c, size_t r) {
		check_range(c,r);return _elems.at(r * n_rows + c);}
	virtual ~Heightmap() {};
private:
	container_t _elems;
	void check_range(size_t c, size_t r) const {
		if (c < 0 || c >= m_columns) throw std::out_of_range("c is invalid");
		if (r < 0 || r >= n_rows) throw std::out_of_range("r is invalid");
	}
};
{% endhighlight %}

Fine and well you say, the meaning is clear, but what is the use of it all?  Ok, let’s develop a more concrete concept that actually does do something.  How about loading the heightmap from an image file, and writing it back?  

Consider the image shown above.  Using my favorite image editor paint.net, I converted this image to an 24-bit BMP file and resized it to be 50 x 50 pixels.  I want a BMP file because this is a file format SDL handles very nicely; the size is an arbitrary choice.  
The fact is, the handy SDL library provides all the heavy lifting we need.  The `SDL_LoadBMP` function creates an SDL_Surface which contains the pixel data in row order.  The pixel data depends on the pixel format declared in the BMP file.  The 24 bits means the data contains 3 bytes per pixel.  Note that the 3 bytes represents an RGB value where R = G = B. Here 0 is black and 255 is white. Clearly, the R, G and B values are not all needed in the matrix: one byte is enough.  

Let us call our new abstraction `HeightmapWithByte`: it is a `Heightmap` with `elemT = unsigned char`.   The code for this abstraction is shown in Listing 2.   The read and write members of this new class template allows us to  swap heightmap data from and to a BMP file.  

**Listing 2:**
{%highlight cpp linenos %}
template <size_t m_columns, size_t n_rows > class HeightmapWithByte
  : public Heightmap<unsigned char, m_columns,n_rows>{
public:
	HeightmapWithByte() : Heightmap<unsigned char,m_columns,n_rows>(0) {}
	void read_from_bmp(const string& bmp_filename) {
		read_heightmap(bmp_filename,
				Heightmap<unsigned char,m_columns,n_rows>::elems().size(),
				Heightmap<unsigned char,m_columns,n_rows>::elems().data());
	}
	void write_to_bmp(const string& bmp_filename) {
		write_heightmap(bmp_filename,
				m_columns,n_rows,
				Heightmap<unsigned char,m_columns,n_rows>::elems().data());
	}
};
{% endhighlight %}

The code for the functions invoked on line 5 and line 11 are shown below in Listing 3.   Both functions assume the the 24-bit per pixel format.  Line 2 refers to a simple wrapper class in GameEx that manages the SDL_Surface.  The pitch of an SDL surface is a notable concept.  It is the number of bytes in a render line. It is the next number after the image width that is divisible by 4.   For example if you have 50 pixels per row, the SDL surface has a pitch is 52.  The consequence is that each row has 2 padding bytes appended in the pixel data of the surface.  So, be extra careful if your fingers itch to unroll that for-loop ;-).  

**Listing 3:**
{% highlight cpp %}
void read_heightmap(const string& filename,size_t buffer_size, unsigned char * buffer) {
	Surface surface(filename);
	SDL_Surface& s = surface.sdl();
	if (s.format->BytesPerPixel != 3)
		throw runtime_error("BMP is not 24 bit.");
	if (buffer_size != static_cast<size_t> (s.w * s.h))
		throw runtime_error("buffer and BMP size does not match.");
	auto source = (unsigned char *) s.pixels;
	for(int c = 0; c < s.w; c++)
		for(int r = 0; r < s.h; r++) {
		      auto pos = r * s.pitch + (c * 3);
		      buffer[r * s.w + c] = source[pos];
		}
}
void write_heightmap(const string& filename,int width, int height, unsigned char * buffer) {
	SDL_Surface * s = SDL_CreateRGBSurface(SDL_SWSURFACE, width, height, 24,
		0x000000FF, 0x0000FF00, 0x00FF0000, 0);
	auto target = (unsigned char *) s->pixels;
	for(int c = 0; c < width; c++)
		for(int r = 0; r < height; r++) {
			auto pos = r * s->pitch + (c * 3);
			target[pos] = target[pos+1] = target[pos+2] = buffer[r * width + c];
		}
	SDL_SaveBMP(s, filename.c_str());
	SDL_FreeSurface(s);
}
{% endhighlight %}

<img src="http://lh4.ggpht.com/-MXua7q56zLI/UdMFTgUOoKI/AAAAAAAAAPI/iJ2Qsv58Lxg/my_hm_thumb%25255B14%25255D.png?imgmax=800" style="float:right"/>

Let's put it all together is a small example program. The snippet in Listing 4 reads the input bitmap, inverts the heights so that the highs are low and lows are high. Then it writes the heightmap to an output BMP file. The image on the right hand side is an enlargement of the output produced by this snippet.  If you compare with the original you'd notice the pixalation effect brought about by the radical transformation of the original input (described above).

**Listing 4:**
{% highlight cpp %}
HeightmapWithByte<50,50> hm;
hm.read_from_bmp("media/terrain/test.bmp");
for_each(e,hm.elems()) {
	*e = 255 - *e;
}
hm.write_to_bmp("my_hm.bmp");
{% endhighlight %}

By the way, that for_each call in the snippet is a very handy macro defined in GameEx.  It works very well with all standard containers, including the new `std::array`.  
There you have it: a few lines of C++ to chew on.  Have fun!
