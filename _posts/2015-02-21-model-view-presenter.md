---
layout: post
category: user interface
tags: [MVP]
---
Many a year ago, in his paper [No Silver Bullet](http://en.wikipedia.org/wiki/No_Silver_Bullet) Fred Brooks separated essential complexity from accidental complexity. The basic argument is that a complex problem requires a design that is complex.  Does this leave us with no hope? I think not.

## The essence of the problem

Complexity in itself is not prohibitive.  When we understand the complexity and we train ourselves to work within it, we become more effective.  There is no silver bullet, but equipped with only a [metal jacket](http://en.wikipedia.org/wiki/Full_metal_jacket_bullet) a trained marine could be a match for that werewolf.

Fighting werewolves is entertaining, but the business of programming has some real challenges and can be much more interesting. 

Little should be said of a program that serves no purpose, so let us consider a typical program that solves a real-world problem.  Not all programs need it, but let's limit ourselves to those programs with a graphical user interface.  Furthermore, let us make the reasonable assumption that the program we have in mind fulfils its purpose only when a human interacts with it. An abundance of questions arise. How should we go about implementing this program? How complex is the essential problem?  Are there ways to manage this complexity?    

At some point before typing in those beautiful lines of source code, it might be a good idea to do some broad stroke designing.  For our envisaged program, there seems to be three essential design elements.  For starters, there is an abstraction of the real-world problem; something I call a _model_.  Then there is a _view_ that renders elements of the model on some kind of graphical device.  Then there are gestures that the user takes in order to interact with the program.  I call these gestures _cues_.  

With an aim to obtain a design that is the least complex, we can apply [Occam's Razor](http://en.wikipedia.org/wiki/Occam%27s_razor) by carefully choosing a good vantage point.  One such vantage point is obtained by answering the right questions in the right order.  First ask: _(1) what is the least complex set of cues needed by the program to fulfil its purpose?_.  Equipped with this set, question two needs an answer: _(2) what is the least complex view required to enable the user to choose a cue?_.  And then, the final design question: _(3) what is the least complex model that will support the required view?_  

This line of reasoning suggests that the complexity of our program is [directly proportional](http://en.wikipedia.org/wiki/Proportionality_(mathematics)) to the complexity of the user cues.  Having fewer or simpler cues lessens the burden on the views and thus your design should end in being a simpler model.  A call this idea _user interface centric design_.  A wise man will combine this design tactic with a [user centered design](http://en.wikipedia.org/wiki/User-centered_design) approach.   

However, a tactic is not a design. What I want to talk about is the [model-view-presenter](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter), a design  pattern that from its name presupposes the existence of a model and hints towards a design in which the view and user cues are first class citizens.  But before we can explore this pattern we need to know more about the problem it tries to solve ([Potel's paper](http://www.wildcrest.com/Potel/Portfolio/mvp.pdf) takes a slightly different angle, but it is a good read on this topic).

## What are the cues?
The user can move a mouse, click on something and type on a keyboard. But creativity abounds in this area: he can also make a gestures on a touch screen, verbalise a command, or simply wave to a [motion detector](http://www.wired.com/2010/11/tonights-release-xbox-kinect-how-does-it-work/all/).  But, it is not medium of input that makes cues complex.  Normally the the input device is outside the program's scope - our program employs the operating system, drivers and libraries that hide away complicated implementation details.

What if we have two possibilities for a cue design? Which cue is less complex than the other?  The answer lies partly in the information that the user needs to covey when taking a particular cue.  Cues that convey less information are less complex.  For example: consider a cue where the user types in text as an alternative to a him clicking on a button.  Assume both has the same objective, which one conveys more information?  It is the text because it conveys a set of characters; while the click conveys an action and a screen coordinate.  The text input is more complex because it must be parsed, and the view must include a facility to show these parse errors.  So, clicking a mouse is likely to be a simpler cue than typing a bit of text.

Another source of complexity for the cue is the information the program needs to respond to it.  Let us call the object that encapsulates the response a _command_.  Consider for example a cue to delete something.  Here the user conveys his intent (i.e. he wants to delete), but we also need the subject that must be deleted.  Let us call the subject a _command parameters_.  

Complex command parameters does not mean there is a complex cue.  There could be sequence of simple cues that the user employs to specify parameters and then another simple cue to initiate the command.  Thus there could be value in considering two types of cues: a _selector cue_ sets up a command parameters and _command cues_ initiates a commands.  

It would be more friendly if our application does not clear the work done by the user after each command is executed.  Typically the user can toggle switches that influences the behaviour and he uses the same selected objects to do other commands.  Obviously he makes this selection on the view.  The things he selects and toggles are concepts from the model.  

We conclude that the cues impose a need on the views to have something the user can click and can select; and that _that_ something has a concept described in the model.  But crucially: a design that recognises the objects that are selected in the view as a state in itself is simpler.  We call this state, the _selection_.

## What is the view?
From a better understanding of cues, it is clear that the view is the thing on which the user makes a selection and initiates an action.  Essentially it does not have to be visual. For example an application that responds to audio commands only also needs the view concept. But here we are focussing on those applications with a graphical device.

The view is not one thing: it is a composition of graphical elements.  Most likely there are buttons and menus that initiates commands, lists or trees of objects that can be selected, toggle button and that shows the state of boolean switches and so on.  

In addition to showing selection state, the view is (most of the time) directly responsible for delivering the program's purpose.  The purpose of many programs is to provide information. We call the graphical elements that performs this function _information views_.

Thus we now have an example of _design feedback_.  The design of an information view may now introduce cues that changes the way information is rendered.  Do not be fooled: these cues are as important as any primary cues.  These information views are likely to become concepts in the model as well.

## What's the model?
Do we understand the model now?  It is a collection of concepts that must be implemented.  The model has a concept for selection; a concept for each element that can be selected, it is likely to have a concept for every information view.   

The model is also responsible for _persistence_.  During his interaction with the program, the user creates new instances of concepts and updates existing instances -- and he might want very well these changes to be permanent. The model is responsible for writing data to  storage and reading data from that storage.

It is good to consider the selection as a special part of the model's design.  For one thing: selection refers to existing concepts.  An we might not need commands to update the selection; thereby simplifying things a bit.

## What is the presenter?
The presenter in the model-view-presenter design pattern glues together the cues, the view and the model.  When the user takes a cue, the presenter decides which command to execute and which view elements to create.  The model elements needed by a view are passed on to it by the presenter. For example: every program has a user cue for _start-up_; and it is the presenter's job to decide what the initial view should look like.

In our design we can start with one presenter element -- let us call it the _application_.  This presenter needs to know about all the commands and views.  Such wide coupling can become cumbersome for large programs, and good design practice should lead to breaking up the presenter into separate concerns.

If we take this separation of concerns to its ultimate breakdown; we could end up with a presenter element for each view element.  The application presenter can create every one of these smaller presenters on start-up.  In fact, this is the kind of design that leads to the concept of a _widget_ (see Bower et. al.'s paper titled [Twisting the triad](http://www.object-arts.com/downloads/papers/TwistingTheTriad.PDF)). Typically a widget spans a continuous region on the display (normally a rectangle)

In a way, a widget is a presenter element because it brings together the user cues, a (generic) model and a graphical view into one unit. But widgets are to be instantiated and something must respond to the _events_ it emits when cues are taken.

Having more than one presenter is likely but one for every view element is too many. Consider a presenter element, which I simply call a _presenter_, as something that encapsulates a number of view elements and their related responses.   

## In a nutshell: what is MVP?
In the model-view-presenter pattern, the presenter observes view elements and executes commands.  Commands may update the model and as a consequence, the presenter may construct new view elements and destroy existing ones. Some commands update the selection: there may be special handling for those -- i.e. bypassing the presenter.  View elements observe the model directly: they reflect changes to the model without being notified by the presenter.
