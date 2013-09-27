---
layout: post
category: graphics
tags: [TBD]
excerpt: TBD 
---
# Background reading
## [Per vertex light](http://www.arcsynthesis.org/gltut/Illumination/Tutorial%2010.html#d0e9855)
The light is computed initially in world space, then transformed into camera space. The camera-space light position is given to both of the shaders. Rendering proceeds normally from there.

## [Another Walch 2005](http://www.gamedev.net/page/resources/_/technical/graphics-programming-and-theory/normal-computations-for-heightfield-lighting-r2264)
### Mean Weighted Equally
In 1971 Gouraud released a book titled "Continuous Shading of Curved Surfaces." In the book Gouraud describes the most common method for computing normals we will refer to his algorithm as "Mean Weighted Equally" (MWE). In essence, if you were to compute the normals of each of the facets of a polyhedron then you could get relatively smooth shading by then taking all of the facets which are attached to a single vertex and averaging the surface normals. Each surface normal contributes equally to the vertex normal.

### Mean Weighted by Angle
"Mean Weighted by Angle" (MWA), suggested that facets which are "attached" to a vertex normal should have their contribution to the vertex normal weighted by the angle of the triangle that the vertex is part of. In other words, all three vertices of a triangle have the same surface normal, however, the surface normal can be multiplied by each of the angles of the triangle in order to create three new normals which are weighted versions of the surface normal. These new normals could then be used to compute vertex normals in place of the surface normals used previously.

## Method 1: Mean Weighted Equally (MWE)
Surface normal: \\( N_s = A \times B \\)
<img src="http://images.gamedev.net/features/programming/normalheightfield/image004.gif" style="float:right"/>

Vertex normal: \\( N_v = \sum{N_s} / 6 \\)

## [Jeromy Walch gamedev article 2013](http://www.gamedev.net/page/resources/_/technical/graphics-programming-and-theory/efficient-normal-computations-for-terrain-lighting-in-directx-10-r3313)

###Slope Method of Computing Heightfield Normals
The vertices of a [heightmap](http://en.wikipedia.org/wiki/Heightmap) are always evenly spaced and never overlap. This makes it possible to break our 3-dimensional heightfield into two, 2-dimensional coordinate systems, one in the XY plane, and one in the ZY plane. We can then use the simple and well-known phrase “rise above run” from elementary geometry to compute the x and z component normals from each of our coordinate systems, while leaving the y-component one. Consider the line shown in figure 3.

## [OpenGL Terrain tutorial](http://www.videotutorialsrock.com/opengl_tutorial/terrain/text.php)

