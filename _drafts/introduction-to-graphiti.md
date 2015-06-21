---
layout: post
category: architecture
tags: [modeling,EMF]
excerpt: What is Graphiti all about?
---

Graphiti enables rapid development of diagram editors for domain models.  Although it is based on EMF (see a <a href="{% post_url 2015-02-21-model-view-presenter%}">previous post about EMF</a> for more information), Graphiti can also work with any Java based domain objects.  A useful feature is that it also includes automatic layout facilities.  

Graphiti is an alternative to [GMF](https://www.eclipse.org/modeling/gmp/).  The key difference is that GMF uses code generation as a key architectural concepts, whereas Graphiti works against Java interfaces.

The easiest way to install Graphiti is to use the [update site](https://www.eclipse.org/graphiti/download.php).

## Building blocks
Graphiti has an *interaction component* that receives user cues (resize drag-and-drop and so on).  These requests are processed by the `diagram type agent`.

The *rendering engine* displays the current data on the screen.

The primary task of the `diagram type agent` is to modify the model data.  The model itself has two components: a `pictogram model` is provided by Graphiti, and you provide the `domain model`.  It has advantages if the domain model is done in EMF.
The two model components are synchronised via *update features*.  There is also a `link model` that is responsible for connections between elements from the `domain model` and the `pictogram model`

The `diagram type agent` is at the heart of a Graphiti based application.  While responding to user cues, it creates (or updates) elements in the `domain model` and elements in the `pictogram model`; as well as the links between them (i.e. the `link model`)

Internally the agent implements `features`. There is a `feature provider` that supplies the list of features for a given context.  These features are the oprations that the end0user can perform on the model.

## How to use Graphiti

The first step is to create a plug-in project using the *Plugin with Graphiti Editor* template.  This template is not available to for stand-alone applications.  

The wizard creates a `DiagramTypeProvider` and a `FeatureProvider`, along with their declarations in  `plugin.xml`.  The wizard is very useful when you have a simple diagram that contains objects of one type and allows links between those types.  If you do not have such a simple model, you can still use the wizard as a statring point and replace the code as you start implementing your own Graphiti editor.  I suspect the wizard is likely to improve in the future.  


## TODO
https://www.eclipse.org/forums/index.php/t/236324/

## References
  * [Graphiti](https://www.eclipse.org/graphiti/)
    * [Getting Started](https://www.eclipse.org/graphiti/documentation/gettingStarted.php)
  * Graphiti developer guide (Eclipse Help File)
