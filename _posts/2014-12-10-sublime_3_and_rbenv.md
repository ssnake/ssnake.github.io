---
layout: post
title: Enable rbenv in Sublime 3
comments: true
tag: sublime 3, rbenv, ruby
---


I’m using Sublime 3 for my rails projects. There is annoying issue with it, it doesn’t work with rbenv out-of-box. If you open ruby file in Sublime 3 and try to build it with ruby build system you may face `ruby command not found` issue. 

To overcome this I created user build system.

Click `Tools | Build System | New Build System`.

Copy following text:

{% highlight console %}
{
	"cmd": ["/home/snake/.rbenv/shims/ruby",  "$file"],
 	"file_regex": "^(...*?):([0-9]*):?([0-9]*)",
    "selector": "source.ruby"
}

{% endhighlight %}

Here is description of values ([full doc](http://sublimetext.info/docs/en/reference/build_systems.html)):

* `cmd`: Array containing the command to run and its desired arguments.

* `file_regex`: Regular expression to capture error output of cmd.

* `selector`: Used when `Tools | Build System | Automatic` is set to true. Sublime Text uses this scope selector to find the appropriate build system for the active view.

Replace `cmd` string with your correct path to ruby. To make it quicker run it in console:

{% highlight console %}
which ruby
{% endhighlight %}

Save file with name like `my_ruby.sublime-build`. `my_ruby` name will appear among build system names. Enjoy!

###Update
if you observe .fuse_hidden* files at sidebar and it annoys you you can easily hide them. Open `Settings-user` file and this string:

{% highlight json %}
{
	"file_exclude_patterns": [".fuse_hidde*"]
}
{% endhighlight %}


