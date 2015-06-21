---
layout: post
category: architecture
tags: [modeling,EMF]
excerpt: What is GMF all about?
---
## What is GMF?
[GMF](https://en.wikipedia.org/wiki/Graphical_Modeling_Framework) brings together EMF <a href="{% post_url 2015-06-20-introduction-to-emf%}">(previously posted)</a> and GEF for developers that want to build diagramming tools.  We call the users of these tools *practitioners* and those that develop the tools, we call *toolsmiths*.  Thus, GMF is a tool for toolsmiths; and it produces an Eclipse plug-in.  

The toolsmith roughly follows these steps:
  1. Create a *GMF Project*. This is the easiest step after you installed GMF.  The easiest way to install GMF is to add the *GMF Tooling* plug-in to your eclipse.
  2. Specify a domain model in an *ecore file*.  This model is described in a <a href="{% post_url 2015-06-20-introduction-to-emf%}">previous post</a>
  3. Specify a graphical definition in a *gmfgraph file*.  This describes the elements used for representation during editing
  4. Define the tooling in a *gmftool file*.  This file is optional and it is used to design the palette, menus, tool-bar and so on.
  5. The elements in the three files described above are combined in mapping model supplied in the *gmfmap file*
  6. Then a generator model is defined in a *gmfgen file*.  Here you define specific implementation details
  7. And now a diagram plug-in is generated. Along with this diagram a *notation model runtime* must be provided that bridges the notation and the domain model.

  There is a handy view called the *GMF Dashboard* that has action in it that helps you with these steps.

## Graphical Definition Model
  Once you are done with the domain model, you can click on the *derive* action in the GMF Dashboard to create the graphical definition model, also called a `gmfgraph`. When the wizard start you have to select the `container`: that is the `eclass` that represents you diagram.

  Then you decide which elements are going to be objects in your diagram and whic are links.  You also choose the propery that would be used as the label for the element.

  TODO: links are properties or instances?


## Tooling Definition Model
  Getting the tooling definition also follows easily from the domain model when using the *derive action*.

  You again have to select the *diagram*, the *elements* and the *links*.  This specified what is going to be on the tool palette.

## Mapping Definition
  Here we click on the *combine action* and accept the wizard defaults.  This is the heart of your GMF project

## Code generation  

## Where to from here?
  TODO Have a look at Sirius

## References
  * Eclipse Wiki
    *  [GMF Section ](https://wiki.eclipse.org/Graphical_Modeling_Framework)
      * [GMF Tutorial](https://wiki.eclipse.org/Graphical_Modeling_Framework/Tutorial#Get_started)
        * [GMF Tutorial Part 1](https://wiki.eclipse.org/Graphical_Modeling_Framework/Tutorial/Part_1)
      * [GMF Documentation](https://wiki.eclipse.org/GMF_Documentation)
# Glossary
  * [Eclipse plug-in](https://wiki.eclipse.org/FAQ_What_is_a_plug-in%3F)
  * [EMF](https://eclipse.org/modeling/emf/)
  * [GEF](https://eclipse.org/gef/)
  * [GMF](https://www.eclipse.org/modeling/gmp/)
  * [GMP](https://www.eclipse.org/modeling/gmp/)
