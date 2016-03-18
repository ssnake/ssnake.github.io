---
layout: post
title: Background processes in production mode
comments: true
---

Intro
========

Working on [sTracker](//stracker.cc) I got to a situation when I needed to run background processes. In particular It was necessary to check tracking numbers time to time. The checking included querying post services and saving received information to DB.
As may you know the trendy solution is to use [DelayedJob gem](https://github.com/collectiveidea/delayed_job) and that's what I made and it's have been working perfectly. So I won't be original to tell you about my steps in order to get DJ working.

DJ
==
## Foreman

So, DJ needs to be started. It can't start itself. For these purposes I used for
 is using background processes. One is clock process and another is delay_job process. I’m using foreman gem to put things together. Here is my solution how to run it in production mode.
My Proc file looks like this:

{% highlight console %}
snake@userv:/opt/projects/stracker$ cat Procfile 
worker:	bundle exec rake jobs:work
clock:		bundle exec clockwork lib/clock.rb
{% endhighlight %}

Make sure foreman is installed on your server. And you have rbenv-sudo installed if not then launch 
git clone git://github.com/dcarley/rbenv-sudo.git ~/.rbenv/plugins/rbenv-sudo

We need to have these processes be up after server is booted. Ubuntu server has upstart deamon which work with scripts in /etc/init folder. You can come up with own script. However, foreman has export command which may help you
rbenv sudo foreman export upstart /etc/init -a stracker -u snake
Where: 
rbenv sudo  - start ruby utility in root mode
foreman export - launch foreman utility with export param
upstart - export into upstart script
-a stracker - name of your application
-u snake - username under which all processes will  be started
Upstart scripts will be created. They are /etc/init/stracker*.conf.


Here is an example of delayed_job script. 

start on starting stracker-worker                                                                                                              
stop on stopping stracker-worker                                                                                                               
respawn                                                                                                                                        
                                                                                                                                               
env PORT=5000                                                                                                                                  
env RAILS_ENV='production'                                                                                                                     
env PATH='/home/snake/.rbenv/shims:/home/snake/.rbenv/versions/2.1.0/bin:/home/snake/.rbenv/libexec:/home/snake/.rbenv/plugins/rbenv-sudo/bin:/
env SECRET_KEY_BASE='2874a29ab2df71f699e58c181356e57bc9c993921ef99eba8776d5286adc7f096a272865abb08d327accd2b59a1dd5936c6f17d441df6c2a39ce012860
env HOME='/opt/projects/stracker'                                                                                                              
                                                                                                                                               
setuid snake                                                                                                                                   
                                                                                                                                               
chdir /opt/projects/stracker                                                                                                                   
                                                                                                                                               
exec bundle exec rake jobs:work 

You may spot env HOME. It’s needed if you use rb-readline gem.
Note: if you use dotenv gem all variables in .env will be added automatically into an upstart script. 

Let’s try to start our scripts. 
	start stracker
where start is alias for initctl start(upstart utility). There are other console commands:
 stop stracker
 status stracker. 
Check logs in /var/log/upstart/stracker-*.log files.


ThreadsPad
==========

