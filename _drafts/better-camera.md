---
layout: post
category: graphics
tags: [TBD]
excerpt: TBD
---


http://www.arcsynthesis.org/gltut/Illumination/Tutorial%2009.html
Mouse Movement
This is the first tutorial that uses mouse movement to orient objects and the camera. These controls will be used throughout the rest of
this book.
The camera can be oriented with the left mouse button. Left-clicking and dragging will rotate the camera around the target point. This will
rotate both horizontally and vertically. Think of the world as a sphere. Starting to drag means placing your finger on the sphere. Moving
your mouse is like moving your finger; the sphere rotates along with your finger's movement. If you hold Ctrl when you left-click, you can
rotate either horizontally or vertically, depending on the direction you move the mouse. Whichever direction is farthest from the original
location clicked will be the axis that is rotated.
The camera's up direction can be changed as well. To do this, left-click while holding Alt. Only horizontal movements of the mouse will
spin the view. Moving left spins counter-clockwise, while moving right spins clockwise.
The camera can be moved closer to it's target point and farther away. To do this, scroll the mouse wheel up and down. Up scrolls move
closer, while down moves farther away.
The object can be controlled by the mouse as well. The object can be oriented with the right-mouse button. Right-clicking and dragging
will rotate the object horizontally and vertically, relative to the current camera view. As with camera controls, holding Ctrl when you right-
click will allow you to rotate horizontally or vertically only.
The object can be spun by right-clicking while holding Alt. As with the other object movements, the spin is relative to the current direction
of the camera.
The code for these are contained in the Unofficial SDK's GL Util library. Specifically, the objects glutil::ViewPole and glutil::ObjectPole.
The source code in them is, outside of how FreeGLUT handles mouse input, nothing that has not been seen previously.
