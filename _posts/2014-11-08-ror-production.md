---
layout: post
title: How to set up RoR application for production mode
---
Hi everyone,

Here is my experience of how to set up your rails application for production mode.
First of all I must admit that this is my first deployment. 

{% highlight ruby %}
Note: All operations were conducted on just installed ubuntu-server 14.04 as a virtualbox machine.  So, this post might be useful for those who want to deploy a rails app on its own server instead of pushing it to heroku, for example. 
{% endhighlight %}
After hard days of development and tests I decided that my sTracker application is ready to be presented to the world. At this point I have the application which is working well in development mode. 

Rails
Suppose we have latest sources on ubuntu server in /opt/projects/stracker folder. We need to figure out if everything work well. We have the working application on our system, but it may not work on a new environment. The best way to check what is broken is to conduct tests.
cd stracker
snake@userv:/opt/projects/stracker$ rake test
The program 'rake' is currently not installed. You can install it by typing:
sudo apt-get install rake

As expected, since the system is virgin, there is nothing except standard packages. We have to install ruby. As RoR site suggests the best practice is using rbenv. Here is the tutorial how to install rbenv (https://github.com/sstephenson/rbenv#installation). My steps are:

git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
Now it’s time to restart my shell so that PATH changes take effect. Usually I do just like this:
	bash
Check if everything is ok:

snake@userv:/opt/projects/stracker$ type rbenv
rbenv is a function
...

Now we have rbenv is installed. I recommend to install two plugins which will make our life easier. We need ruby-build plugin and rbenv-sudo:
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build  
git clone git://github.com/dcarley/rbenv-sudo.git ~/.rbenv/plugins/rbenv-sudo
Now it’s time for ruby itself. We will install ruby 2.1.0, during the process of installation ruby may need some developing packages. Here is list which may help you:
sudo apt-get install gcc g++ make libssl-dev sqlite3 libsqlite3-dev
rbenv install 2.1.0
And set 2.1.0 as global version and install bundle and rails
rbenv global 2.1.0
gem install bundle
gem install rails
Note: After installing some gems you may see that they don’t work from command line. To overcome this you have to restart shell.
Now make sure we are in our application folder. We will install all gems are necessary for the app.
	bundle install
Now test it if all is up and ok
	rake test
Hope everything is ok.
	rake db:setup
	rails server
Now we should have our fully working  application. Check it via web browser.



Web server
Setting up the application to run in production mode is pretty challenging. The biggest challenge is to setup web server. We will deploy the app using apache and phusion passenger (https://github.com/phusion/passenger). Passenger it’s kind of replacement of WEBrick. It’s web server and as it stated on its web site: It is designed to be easy to use, fast, stable and reliable and it’s written in c++. 

First of all, we need to install Passenger gem. 	
gem install passenger
After restarting shell, you should see passenger’s help tool like passenger-install-apache2-module.
Start it and it will guide you via process of integration passenger into apache.
	 passenger-install-apache2-module
Note: During this process you may face “Your system does not have a lot of virtual memory” error. So, make sure you virtualbox machine has more than 512mb of memory.


Please edit your Apache configuration file, and add these lines:
At the end of successful compilation process you should get :
   LoadModule passenger_module /home/snake/.rbenv/versions/2.1.0/lib/ruby/gems/2.1.0/gems/passenger-4.0.52/buildout/apache2/mod_passenger.so
   <IfModule mod_passenger.c>
     PassengerRoot /home/snake/.rbenv/versions/2.1.0/lib/ruby/gems/2.1.0/gems/passenger-4.0.52
     PassengerDefaultRuby /home/snake/.rbenv/versions/2.1.0/bin/ruby
   </IfModule>

Suppose you have a web application in /somewhere. Add a virtual host to your
Apache configuration file and set its DocumentRoot to /somewhere/public:

   <VirtualHost *:80>
      ServerName www.yourhost.com
      # !!! Be sure to point DocumentRoot to 'public'!
      DocumentRoot /somewhere/public    
      <Directory /somewhere/public>
         # This relaxes Apache security settings.
         AllowOverride all
         # MultiViews must be turned off.
         Options -MultiViews
         # Uncomment this if you're on Apache >= 2.4:
         #Require all granted
      </Directory>
   </VirtualHost>

First paragraph is the configuration for passgner_module. Let’s add it to apache. Go to /etc/apache2/mods-available folder and create two files: passenger.load and passenger.conf.
Add   “LoadModule passenger_module ...mod_passenger.so” string into passenger.load and add remaining text “<IfModule mod_passenger.c>...  </IfModule>” to passenger.conf.
Ok files are added, now we will enable them:
sudo a2enmod passenger
	sudo service apache2 restart
Make sure there are no errors.
It’s turn for site configuration. Go to /etc/apache2/sites-available, create st.conf and add following text:
<VirtualHost *:80>
      #userv is hostname of my virtualbox pc
      ServerName userv
      #this is path to my RoR app
      DocumentRoot /opt/projects/stracker/public
      <Directory /opt/projects/stracker/public>
         # This relaxes Apache security settings.
         AllowOverride all
         # MultiViews must be turned off.
         Options -MultiViews
         # Uncomment this if you're on Apache >= 2.4:
         Require all granted
     </Directory>
</VirtualHost>



Enable site:
	sudo a2ensite st.conf
	sudo service apache2 restart

Now check if site is up, input “userv”(hostname of our virtualbox pc) into your browser and see what you get. At this point I get “500 Internal Server Error". Despite the fact that we followed all instructions passenger helper provided us it doesn’t work. 

Look at /var/log/apache2/error.log:
[ 2014-10-03 11:15:05.2952 4807/7fb4b79e5780 agents/Watchdog/Main.cpp:728 ]: All Phusion Passenger agents started!
App 4888 stdout:
App 4888 stderr: /usr/bin/env:
App 4888 stderr: ruby
App 4888 stderr: : No such file or directory
App 4888 stderr:

Passenger can’t find ruby. The folder we pointed in passenger.conf is correct and everything should work...To overcome this issue open passenger.conf and replace current
PassengerDefaultRuby /home/snake/.rbenv/versions/2.1.0/bin/ruby
with 
PassengerDefaultRuby /home/snake/.rbenv/shims/ruby
Restart apache and check if error goes away.

Refresh your browser and check if our app is working. 

Mine doesn’t work. I faced missing secret_key_base issue:

App 4888 stderr: [ 2014-10-03 11:15:15.5546 4919/0x007f2746d71008(Worker 1) utils.rb:84 ]: *** Exception RuntimeError in Rack application objec
t (Missing `secret_key_base` for 'production' environment, set this value in `config/secrets.yml`) (process 4919, thread 0x007f2746d71008(Worke
r 1)):

If you look at config/secrets.yml you may see this:
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>

 
All we need is to add secret key into environmental value SECRET_KEY_BASE. 
I found out for myself two ways to do this in elegant way.
Using Profile.d
All wide-system variables could be added into /etc/profile.d. To generate secret key we need be inside our app folder and launch rake secret command. Then generated key should be added into .sh script in profile.d folder. To do it in one command line string input following:
echo "export SECRET_KEY_BASE=`rake secret`" | sudo tee /etc/profile.d/st_env.sh 

To made SECRET_KEY_BASE available without rebooting just restart your shell. 

Note: Anything you put in /etc/profile.d/ will be run, every time you open a login shell.

Using dotenv-deployment gem
There is a useful gem dotenv which work with .env file. In this file you can initialize any variables. They will be added into environment automatically when your app is started. 
There are two versions of this gem. Dotenv gem is mainly used in developing mode. For production mode there is dotenv-deployment gem.

Ok, at this point we have SECKET_KEY_BASE added. Refresh your browser. You should see you RoR application. If not, check log/production.log
Unfortunately  I faced two problems. First, I forgot to setup my DB and second CSS styles is not working.
To deal with first issue, type:
 
RAILS_ENV="production" rake db:setup
And for second one:
RAILS_ENV=”production” rake assets:precompile
Reload apache and try your browser again. Hope now it works flawlessly


Background processes in production mode
My app is using background processes. One is clock process and another is delay_job process. I’m using foreman gem to put things together. Here is my solution how to run it in production mode.
My Proc file looks like this:
snake@userv:/opt/projects/stracker$ cat Procfile 
worker:	bundle exec rake jobs:work
clock:		bundle exec clockwork lib/clock.rb

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








