---
layout: post
category: user interface
tags: [WTL]
---
In this post we explore a minimal approach to running a job: we show a [modal dialog](http://en.wikipedia.org/wiki/Modal_window) which we call a ProgressDialog.  This dialog shows how the job is progressing; and allows the user to cancel the job.


It happens sometimes that a [desktop application](http://www.pcmag.com/encyclopedia/term/41158/desktop-application) starts a  _job_ that can take some time to complete.  This job should not hold up the _user interface thread_; if it does the application becomes unresponsive until the job completes.  

## The general idea
Let us call the thread that starts the `Job`, the _parent thread_.  The worker thread sends a `JobReport` to the parent thread, and the parent thread can call the `Job::cancel()` [member function](https://msdn.microsoft.com/en-us/library/fk812w4w.aspx). The client observes the job for updates.

A problem arises because the client receives the `JobReport` on the worker thread; but the window update must be done on the parent thread.  Lucky for us, [SendMessage](https://msdn.microsoft.com/en-us/library/windows/desktop/ms644950%28v=vs.85%29.aspx) solves this problem nicely. It switches to the thread of the window for which it is called.  A minor disadvantage of this approach is that the worker thread is blocked while the window is updated. (Note that the Windows API has alternative options; I am just keeping things simple here).

A second problem to be solved is the exchange of progress data between the worker and the parent thread.  For this type of problem the [std::atomic](https://msdn.microsoft.com/en-us/library/hh874651.aspx) class is a good solution.  The data to be exchanged is in the `JobReport` -- the worker [stores](https://msdn.microsoft.com/en-us/library/hh874758.aspx) it and the client [loads](https://msdn.microsoft.com/en-us/library/hh874622.aspx) it.

Another data exchange is the _cancel_ message sent from another thread to the client thread. For this we use a `std::atomic<bool>` called `is_cancelled` -- this value is stored by the client (probably via the parent thread) (via `Job::cancel()`) and loaded by the worker thread.

## User interface elements
We have to look no further than the [CDialogImpl](https://msdn.microsoft.com/en-us/library/79bke8xf.aspx) class to use as the [base class](http://dictionary.reference.com/browse/base+class) for the `ProgressDialog`.  Following the model view presenter approach introduced in <a href="{% post_url 2015-03-01-model-view-presenter-primer %}">the previous post</a>, we create a `ProgressView` that manages the dialog.

However, we deviate slightly here from the previous pattern because the view is a dialog.  It cannot be referenced by other views and it is model.  The best way to do this is with a static method called `run()` shown below:

{% highlight cpp %}
/* returns false when the user pressed cancel */
bool ProgressView::run(Job & j) {
	ProgressDialog d(j);
	Job::observer_ptr obs = j%[&d]() {
		d.update();
	};
	auto r = d.DoModal();
	j.wait_for_complete();
	return r != IDCANCEL;
}
{% endhighlight %}
