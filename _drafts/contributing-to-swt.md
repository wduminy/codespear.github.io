---
layout: post
category: programming
tags: [SWT, Open Source]
excerpt: Attempt to contribute to SWT
---

Getting the development environment up and running is the first step.  
The docs suggest that the basic version is fine; but it is better to download the eclipse for rcp developers.
Basically you need to download the basic version of Eclipse and these to repositories:

 - https://github.com/eclipse/eclipse.platform.swt.binaries
 - https://github.com/eclipse/eclipse.platform.swt
(Do we need both?)
The [using SWT from Git page](https://www.eclipse.org/swt/git.php) on the Eclipse site describes the set-up process in some detail.

Current problem: I think if you have 64 bit windows you have to get 64bit jdk; not something I would like to do at the moment -- so sad!

** Stuff **
Before you can be accepted you must sign the [Eclipse Foundation Contributor License Agreement](http://www.eclipse.org/legal/CLA.php).  A relatively simple process.



The [developer page](https://projects.eclipse.org/projects/eclipse.platform.swt/developer) mentions that they use Gerrit.  But there is little help.  

The [SWT project page](https://www.eclipse.org/swt/) is a little more help, especially the [tools page](https://www.eclipse.org/swt/tools.php).  There is also more of on the [how to fix a bug page](https://www.eclipse.org/swt/fixbugs.php)
 - Use the most basic eclipse
 - [JniGen](https://www.eclipse.org/swt/jnigen.php)
 - [Sleak](http://www.eclipse.org/articles/swt-design-2/sleak.htm)
 - SWT Spy plug-in

There there is some information about [using SWT from GIT](https://www.eclipse.org/swt/git.php) 