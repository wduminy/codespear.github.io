---
layout: post
category: programming
tags: [user interface, WTL]
excerpt: The humble file path program
---

In a previous post about the 
<a href="{% post_url 2015-02-21-model-view-presenter%}">model-view-presenter</a> I suggested a way design a program that has a user interface.  Here we make things a bit more practical by exploring how we can write such a program using WTL.  

Here is our example problem:  The user wants to capture a list of folder names and store that list.  Every item in the list is string if text, but the text must identify a reachable file location.  We now have the beginnings of a simple program with a spurious purpose. 

## Basic program design
Starting with cues: 
 * (C1) Create new item in folder list, 
 * (C2) delete items from folder list and 
 * (C3) change the text of an item.  

This leads to view elements 
 * (V1) a [list control](https://msdn.microsoft.com/en-us/library/bycfwcsh.aspx) to show the paths,
 * (V2) a context menu item for _new item_,
 * (V3) a context menu item for _delete_ and 
 * (V4) a status bar to show how many items are selected. 

These view elements in turn lead to model elements: 
 * (M1) a `FilePath` that captures the text for one item, 
 * (M2) a `Project` that contains a list of folder items and 
 * (M3) a `Selection` that identifies which items are selected. 

The model will be persisted to a `*.prj` file.  Persistence leads to new elements:
 * (C4) Load exiting folder list,
 * (C5) save folder list,
 * (V5) a main menu item for _loading_ and
 * (V6) a main menu item for _saving_ the project.

For this simple program we are going to stick to one presenter which we will call `Application`.  

## The start-up sequence
According to model-view-presenter we want our presenter, the `Application` to render the first view when the application starts up.  This initial view will show an empty `Project`.

But how do we code this start-up sequence using WTL?  The WTL wizard creates a good start.  However it is very clear in the generated `Run` method the `CMainFrame` runs the show.  In a way this is true, but we need to the the `Application` in these somehow.

A simple way to do this is to make `wndMain` a property of the application. 

## The view 





The  has a name in WTL that is different from its MFC counterpart.  In MFC this control is called `CListCtrl` while WTL calls `CListViewCtrl`.  There are different ways to use this class, as described in [Michael Dunn's article](http://www.codeproject.com/Articles/4028/WTL-for-MFC-Programmers-Part-IV-Dialogs-and-Contro#usingwrap).

## Background reading
[AlanW list control vista style](http://www.codeproject.com/Articles/13383/ListCtrl-A-WTL-list-control-with-Windows-Vista-sty)

[Extended Style](https://msdn.microsoft.com/en-us/library/c7ezbf7b.aspx)