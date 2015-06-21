---
layout: post
category: architecture
tags: [modeling,EMF]
excerpt: What is EMF all about?
---

The Eclipse modeling framework ([EMF](https://en.wikipedia.org/wiki/Eclipse_Modeling_Framework)) is a modeling framework and code generation facility for building tools and other applications based on a structured data model. From a model specification described in [XMI](https://en.wikipedia.org/wiki/XML_Metadata_Interchange), EMF provides tools and runtime support to produce a set of Java classes for the model, along with a set of adapter classes that enable viewing and command-based editing of the model, and a basic graphical editor.

The easiest way to install EMF is to use the [all in one update zip file](https://www.eclipse.org/modeling/emf/downloads/) and install this zip file using the usual Eclipse *add plug-in* feature.

You use EMF by roughly following these steps:
  1. Create an *Empty EMF project* using the EMF project wizard.
  2. Create an   [Ecore](https://www.eclipse.org/ecoretools/) model.  Ecore is a modeling language that is a subset of [UML](https://en.wikipedia.org/wiki/Unified_Modeling_Language).  
  3. From this model you generate Java code and an [Eclipse plug-in](https://wiki.eclipse.org/FAQ_What_is_a_plug-in%3F).

## The Ecore Model
  Create the `ecore file` in the `model` directory of the project. The default editor provides a tree-based view of the Ecore model.  Select elements in this editor can be update using the properties view.

  The idea of the model is to build up a structured hierarchy. This structure is called a *containment tree*.  Typically the model has one class that represents an object that contains instances of other classes of the model. Thus you are like to have a class called something like *Diagram* as the root of the containment tree.

  We also need the concept of package.  This concept relates to the Java package and the XML namespace that will be used by the model.

### The package
 This *model package* needs a name and a [URI](https://en.wikipedia.org/wiki/Uniform_resource_identifier).  

 The `name` property is the last part of the Java package name; so might be a good idea to keep it lowercase.  

 The `ns prefix` and `ns uri` properties are used to determine the [XML namespace](http://www.w3schools.com/xml/xml_namespaces.asp) to be used in the XML produced by the user of the application.  

### The EClass
  An `EClass` can be created in a package.  It contains `EAttributes` and `EReferences`. It also has `ESuper Types`, handy if you are capturing the same attributes for every object.

  A good convention is to use uppercase letters for class `names` and [lower camel case](https://en.wikipedia.org/wiki/CamelCase) for attribute `names`.

### The EAttribute
  The most important properties of an `EAttribute` is it's `name` and it's `EAttribute Type`

### The EReference
  An `EReference` is an association between two classes.  If you set `containment` to `true` this means that the reference contributes to the containment tree.

  The value for `UpperBound` is `-1` when you want to specify a *many association*.

## Generating code
  In order to generate the code, you need an *EMF generator model*.  This `genmodel file` be created using the Eclipse wizard.  The wizard validates the model and shows errors when it cannot create the `genmodel file`.

  The generator model is based on the `ecore file`; and it is possible (after updating the model) to reload the `ecore file` into the `genmodel file`.  

  You can now generate the code by invoking the action called `generate all` on the root of the element in the `genmodel`.

  Run your new application by invoking the `Run as Eclipse Application` from the original *EMF Project*.  You can now add a model to of the type you specified in a new project.

  What you now have is a basic tree-structure editor someone can use to capture an instance your model.  This editor application saves the model in an XML file using a format you would expect when considering the elements you defined in your model.

  However, that is not all. EMF also provides the beginnings of a full blown editing application that you can extend and modify.  Have a look at the [EMF Client Platform](https://www.eclipse.org/ecp/) and [Graphiti](https://www.eclipse.org/graphiti/) to see what is possible.

## References
  * Eclipse Projects
    * [EMF](https://eclipse.org/modeling/emf/)
      * [EMF Documentation](https://www.eclipse.org/modeling/emf/docs/)
  * Eclipse Source web site
    * [EMF Tutorial](http://eclipsesource.com/blogs/tutorials/emf-tutorial/)
