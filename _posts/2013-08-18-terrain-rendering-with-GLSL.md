---
layout: post
category: graphics
tags: [GLSL,terrain]
---
In the post about [faster terrain rendering](http://dracocepheus.blogspot.com/2013/07/faster-terrain-render.html) we focused on how you can use buffers to improve the performance of the terrain rendering procedure.  Here we explore a method to render the terrain using [GLSL](http://www.opengl.org/documentation/glsl/).  


<img src="/assets/images/2012/terrain_1.jpg" style="float:right"/>

I wanted to distribute colors in the terrain.  On the hill tops we have ice; and they are white, somewhere below we have a layer of rock that's gray, grass is green and then there is some sand for the beaches.  Instead of clearcut boundaries for the layers, I'd like the colors to blend into each other. The basic idea is that each color has weight relative to the other colors surrounding it.

The effect I wanted is described neatly in the [multi-texturing lesson](http://www.riemers.net/eng/Tutorials/XNA/Csharp/Series4/Multitexturing.php") of Riemer's tutorial series.  It a good idea to read that lesson. Keep in mind that Riemer uses textures, where I use colors, and he uses a different technology.  However, the idea is the same.    

In the listing below You can see the _vertex shader_. It uses a _varying float_ to send a _height_ to the _fragment shader_.  The height is normalized to be within the range (0.0;1.0).

<b>Vertex shader listing:</b>
{%highlight glsl%}
#version 330
varying float height;
void main() {
	gl_Position = (gl_ModelViewProjectionMatrix * gl_Vertex);
	height = (gl_Vertex.z + 1)/10;
}
{%endhighlight%}

The next listing shows the _fragment shader_ that outputs the fragment color ( _gl\_FragColor_ ). The algorithm is very close to the one used by Riemer. Note the the value for _m_ and _u_ is quite dependent on the other values.

<b>Fragment shader listing:</b>
{%highlight glsl%}
#version 330

const vec3 white = vec3(1,1,1);
const vec3 gray = vec3(0.75,0.75,0.75);
const vec3 green = vec3(0,0.5,0);
const vec3 blue = vec3(0,0,1);
const vec3 brown = vec3(0.7,0.6,0.1);
const float ice = 0.94;
const float rock = 0.65;
const float grass = 0.35;
const float sand = 0.06;
const float m = 8.5;   
const float u = 1.6;

varying float height;

float weight(float v) {return clamp(u - abs(height-v)*m,0,1);}

void main(){
	float ice_d = weight(ice);
	float rock_d = weight(rock);
	float grass_d = weight(grass);
	float sand_d = weight(sand);
	float total = ice_d + rock_d + grass_d + sand_d;
	gl_FragColor = vec4(
    	ice_d * white / total +
    	rock_d * gray / total +
    	grass_d * green / total +  
    	sand_d * brown / total
    	, 1);
}
{%endhighlight%}

That's all for this post.
