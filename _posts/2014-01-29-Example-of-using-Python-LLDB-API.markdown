---
layout: post
title:  "Example of using Python LLDB API"
date:   2013-01-29 08:36:29
categories: python lldb
---

There are not much information on LLDB API. Even the official website mentioned during one of the WWDC 2012 sessions,while definitely handy, doesn't provide a whole lot of information. Also, there are a couple of posts over at [clarktech](http://www.libertypages.com/clarktech/?p=4289) on the topic. 

As I've been interested in python lately I decided to try to come up with something useful for my iOS development workflow. One of my current iOS projects has view hierarchy where position of a view is very important. I needed to somehow visualize the hierarchy.



#View Hierarchies

On iOS there are several ways to add a view to view hierarchy:

{% highlight objective-c %}
[self.view addSubview:newView];
[self.view insertSubview:newView belowView:someView];
[self.view insertSubview:newView aboveView:someView];
{% endhighlight %}

After newView has been added using -addSubiew it is available in subviews array. It is the last object in this array and can be accessed using [parentView.subviews lastObject]. So  what If we have several views at each level of hierarchy and need to track which view is on top of which. We can send recursiveDesription message to view (lldb) po [self.view recursiveDescription] or write our own version of this function that limits the amount of info and makes it more sane.

However, as you can see these approaches doesn't provide ivars names of views. So we have to infer what view we are looking at from other information like its type and size. This is not fun.



#LLDB API

{% highlight python %}
node = lldb.frame.GetValueForVariablePath("*self").GetChildAtIndex(newViewIndex) 
{% endhighlight %}

will give us object in self at index newViewIndex and assign it to variable node. When we added a newView to self.view it retained newView along with its variable name. Using LLDB GetName() call we can get the name of the variable. So calling node.GetName() will return "newView".

Now GetChildAtIndex(i) returns an object stored in self at index i. It can be any object. What we need however is a subclass of UIView. Because LLDB can evaluate expressions we can iterate over self children sending the following expression

{% highlight bash %}
'(BOOL)[self->%s isKindOfClass:(id)[UIView class]]' % (node.GetName())
{% endhighlight %}

We can now write names of views and their memory addresses in a dictionary, get subviews recursively similar to methods described in the first section, match those subviews against the dictionary of names by memory address. All that's left is to output the result in a nice format.

#Magic of Python

Because of dynamic typing and pretty much automatic memory management (by the way based on reference counting) writing a python script is a great way to put together a quick solution. I wanted to have expandable view hierarchy rendered as a web page. So I used python marker.py module to construct a webpage and insert the views and subviews as  ul li elements. I also used jQuery to quickly add folding / unfolding and CSS to make the list look nice.


You can download the code from [GitHub](https://github.com/linkov/uiview-hierarchy-generator)