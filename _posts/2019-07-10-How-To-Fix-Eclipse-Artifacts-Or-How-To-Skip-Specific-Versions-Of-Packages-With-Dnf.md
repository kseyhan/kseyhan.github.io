---
layout: post
title: "How to fix menu artifacts of Eclipse on Fedora 30 or how to skip specific versions of packages with dnf"
description: How to fix menu artifacts of Eclipse on Fedora 30 or how to skip specific versions of packages with dnf.
categories: [eclipse]
tags: [eclipse, fedora, gtk3, dnf, downgrade, versionlock, python3-dnf-plugin-versionlock,  gtk3-3.24.9]
---

If you're on a bleeding edge linux distribution like Fedora 30 or Arch you may encounter some ugly artifacts in your programs right now like i do with Eclipse which looks something like this:

![screenshot eclipse with overlapping artifacts in the project explorer](/assets/img/posts/2019-07-10/eclipse.png){: width="400px"}

Doing some research leads us to following Bug entry in the Eclipse Bugtracker.


[https://bugs.eclipse.org/bugs/show_bug.cgi?id=548629](https://bugs.eclipse.org/bugs/show_bug.cgi?id=548629)

Where we can find out that its a problem with the ***gtk version 3.24.9*** and that the devs are working on some fixes for it.

[https://gitlab.gnome.org/GNOME/gtk/issues/1977](https://gitlab.gnome.org/GNOME/gtk/issues/1977)

So how to fix this now?

My solution to fix this problem is as follows.

* Downgrade the gtk version to 3.24.8

* And skip the current version of gtk3 from your updates until the next version 3.24.10 will show up.


So lets go..

{% highlight bash %}
sudo dnf downgrade gtk3-3.24.8
{% endhighlight %}

So far so good we should have working menus in eclipse again:


![screenshot eclipse fixed view](/assets/img/posts/2019-07-10/eclipse-fixed.png){: width="200px"}

but after we run ```dnf update``` the next time gtk3 will automaticly update again.

To deal with this problem we have different solutions like running dnf update with the exclude flag as follows ```dnf update -x gtk3``` or exlcude gtk completely from the package manager or we make it proper by just installing a dnf plugin which will allow us to ***skip a specific version*** of gtk until a new version shows up.

{% highlight bash %}
sudo dnf install python3-dnf-plugin-versionlock
{% endhighlight %}

{% highlight bash %}
sudo dnf versionlock exclude gtk3-3.24.9-*.fc30
{% endhighlight %}

Here we go. We should be able to use dnf like we're used to and the fixed version of Gtk3 should show up as soon as its available through the package manager for us.
