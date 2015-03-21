---
layout: post
category: user interface
tags: [WTL]
excerpt: A dialog to show job progress
---

It happens sometimes that a user starts a  _job_ that can take some time to complete.  This work should happen in a _worker thread_. What happens with the user interface while  the job is running?  A simple approach is to show a [modal dialog](http://en.wikipedia.org/wiki/Modal_window) through which the user can see how things are progressing.  He can also cancel the job using this dialog. 

## Basic idea
Let us call the thread that starts the `Job`, the _parent thread_.  The worker thread sends a `JobReport` to the parent thread, and the parent thread can call the `Job::cancel()` [member function](https://msdn.microsoft.com/en-us/library/fk812w4w.aspx). The client observes the job for updates.

A problem arises because that client receives the `JobReport` on the worker thread; but the window update must be done on the parent thread.  The [PostThreadMessage](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644946%28v=vs.85%29.aspx) function helps -- it uses the parent's _thread id_.. The `Job` obtains the id of the parent thread (i.e. the thread on which it is constructed), using [GetCurrentThreadId](https://msdn.microsoft.com/en-us/library/windows/desktop/ms683183%28v=vs.85%29.aspx). Then the worker sends through that id to the observer as a data member of the `JobReport`.   

A second problem to be solved is the exchange of data between the worker and the parent thread.  For this type of problem the [std::atomic](https://msdn.microsoft.com/en-us/library/hh874651.aspx) class is a good solution.  The data to be exchanged is in the `JobReport` -- the worker [stores](https://msdn.microsoft.com/en-us/library/hh874758.aspx) it and the client [loads](https://msdn.microsoft.com/en-us/library/hh874622.aspx) it. Another data exchange is a message from the parent thread to the client thread; for this we use a `std::atomic<bool>` called `is_cancelled` -- this value is stored by the client (probably via the parent thread) (via `Job::cancel()`) and loaded by the worker thread.

## User interface elements
[CDialogImpl](https://msdn.microsoft.com/en-us/library/79bke8xf.aspx)
[atomic_bool](https://msdn.microsoft.com/en-us/library/hh874894.aspx)
[thread](https://msdn.microsoft.com/en-us/library/hh920526.aspx)
[future](https://msdn.microsoft.com/en-us/library/hh920535.aspx)


[code highlighting](http://thanpol.as/jekyll/jekyll-code-highlight-and-line-numbers-problem-solved/)

[CreateThread](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682453%28v=vs.85%29.aspx)


