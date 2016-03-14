---
layout: post
title: HowTo Compile Ruby under Windows
comments: true
tag: ruby, windows
---

For many of us ruby running on windows is a pain, but there might be cases when you have to overcome this pain and install ruby on windows.



Here is my steps which solved following issues I faced. It was tested on Windows XP and Windows 2003(x86):

* securerandom.rb:156 segmentation fault

* The procedure entry point _gmtime64_s could not be located in the dynamic link library msvcrt.dll.

* SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed


## Before start 

You have to install any working ruby version, becausue following RubyInstaller script is worked via `rake`.

Here is link for working installer which doesn't have "entry point" bug.

[Ruby 2.0.0](http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.0.0-p598.exe?direct)

## RubyInstaller Dev Kit

First of all, git clone the Rubyinstaller Dev kit.


{% highlight console %}
git clone git://github.com/oneclick/rubyinstaller.git
cd rubyinstaller
{% endhighlight %}
It's rake-based app. You can check its command by `rake -T`.


## Dev Kit

Now we need to install dev kit


{% highlight console %}
rake devkit sfx=1 dkver=mingw-32-4.6.2
{% endhighlight %}
Using mingw-32-4.6.2 allowed me to solve "segmentation fault".
After compilation you will have something like `pkg\DevKit-mingw-32-4.6.2-20150211-0706.7z`
Unpack it to `C:\DevKit`. Make sure `C:\DevKit\bin` and `C:\DevKit\mingw\bin` are in PATH.


## Ruby

Now compile ruby

{% highlight console %}
rake clean
rake ruby21
{% endhighlight %}
If it's successfully compiled the `sandbox\ruby21_mingw` folder will have ruby. Copy them to `C:\Ruby21`.


## SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed


* Download [pem](https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/AddTrustExternalCARoot-2048.pem) file

* Copy this pem file to `C:\Ruby21\lib\ruby\2.1.0\ssl_certs`

[source](https://gist.github.com/luislavena/f064211759ee0f806c88)