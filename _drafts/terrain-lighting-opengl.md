---
layout: post
category: graphics
tags: [TBD]
excerpt: TBD 
---
# Background reading

## [Jeromy Walch gamedev article 2013](http://www.gamedev.net/page/resources/_/technical/graphics-programming-and-theory/efficient-normal-computations-for-terrain-lighting-in-directx-10-r3313)

###Slope Method of Computing Heightfield Normals
The vertices of a [heightmap](http://en.wikipedia.org/wiki/Heightmap) are always evenly spaced and never overlap. This makes it possible to break our 3-dimensional heightfield into two, 2-dimensional coordinate systems, one in the XY plane, and one in the ZY plane. We can then use the simple and well-known phrase “rise above run” from elementary geometry to compute the x and z component normals from each of our coordinate systems, while leaving the y-component one. Consider the line shown in figure 3.

## [Another Walch 2005](http://www.gamedev.net/page/resources/_/technical/graphics-programming-and-theory/normal-computations-for-heightfield-lighting-r2264)
## [OpenGL Terrain tutorial](http://www.videotutorialsrock.com/opengl_tutorial/terrain/text.php)

