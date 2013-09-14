---
layout: post
category: graphics
tags: [terrain]
excerpt: Find a point on the surface of a terrain 
---
In <a href="{% post_url 2013-08-18-terrain-rendering-with-GLSL %}">a previous post</a>, we explored how to render a heightmap based terrain.  Here we look at how we can determine a point on the surface of that terrain.

Recall from the <a href="{% post_url 2013-07-13-terrain-rendering-basics %}">terrain basics post</a> that \\(z\\) points upwards, \\(x\\) points west and \\(y\\) points south.  So using this coordinate system we need to determine the \\(z\\) value, given an \\((x,y)\\) coordinate.

The first thing we need to do is to find the triangle in the heightmap for the coordinate \\((x,y)\\).  Then you find the plane of the triangle, and the line that goes through the coordinates.  The you calculate the point of [intersection between the plane and the line](http://en.wikipedia.org/wiki/Line-plane_intersection).  

## Finding the triangle      

#Backgound reading
# [SO foot contact point](http://gamedev.stackexchange.com/questions/54645/how-to-determine-characters-foot-contact-point-on-a-uniform-triangle-mesh-terra)
*Byte56* says: If you know how your mesh is generated, you can easily use something like linear interpolation to find the height at the contact point. Using the distance to the surrounding points, and their height values, you can find the height value at the contact point.

# [SO heighmap terrain](http://stackoverflow.com/questions/12416195/3d-heightmap-terrain-and-collision-detection)
	I have 3 vertices that define a plane. (the 3 nearest pixels in a height map)
	I have an x,z on that plane. (my location in the world)
	How do you find the y-intercept? (so that I stand on the surface of that plane)

*ebox* says:
The equation of a plane is:
\\( Ax + By + Cz = D \\) , where \\( D = Ax_0 + By_0 + Cz_0 \\),

If you have three vertices, find two vectors from the vertices. For example, for three vertices T, U, V, there would be, a vector \\(TU\\), and a a vector \\(UV\\).

Find the cross product of the two vectors. That's your normal vector, **n**, which has three components n1, n2, and n3.

    A = n1
    B = n2
    C = n3

Take one of the points. The coordinates of that point are x0, y0, and z0.

Input this into the equation to calculate D.

Then substitute your x and z for x and z and solve for y!


So in the end y is:

    y = (A*x0 + B*y0 + C*z0 - A*x - C*z)/B

Somebody correct me if my algebra was wrong.

You can calculate the cross product like this:

For two vectors **a** and **b**, with components a1, a2, a3 and b1, b2, b3, respectively, the cross product is :

![enter image description here][1]

which goes to:

![enter image description here][2]

A = the coefficient of i-hat (the bolded i)

B = the coefficient of j-hat (the bolded j)

C = the coefficient of k-hat (the bolded k)


  [1]: http://i.stack.imgur.com/LfhF6.png
  [2]: http://i.stack.imgur.com/atxmf.png

