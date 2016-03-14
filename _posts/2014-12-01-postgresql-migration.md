---
layout: post
title: How to move Postgre DB from one server to another
comments: true
tag: [postgresql, rails, migration, backup]
---

I need to run my RoR application on a new server.  My application already has some vital data in its DB. I'm using PostgreSQL as DB. All app code is up and in git. There shouldn't be a problem to deploy it to new place. The weak side is DB. I need to move it unchangebly from one place to another. Here is my steps of how to achive this. 

The best way to do it is to backup db on source server and restore it on destination server.
According to [official documentation](http://www.postgresql.org/docs/9.3/static/backup-file.html) there are 3 ways how to do this. I chose `File System Level Backup` way.

Let's log in to source server and find out place of data directory. On source server I have ubuntu distributive and installed PostgreSQL 9.3. Open `/etc/postgresql/9.3/main/postgresql.conf` and find line with `data_directory`. My line is `data_directory = '/var/lib/postgresql/9.3/main'`.  
Backup it via following command:

{% highlight bash %}
sudo tar -cf backup.tar /var/lib/postgresql
{% endhighlight %}
Log in to destination server and install PostgreSQL.

{% highlight bash %}
apt-get install postgresql-9.3 postgresql-server-dev-9.3
{% endhighlight %}
You may see following message:

{% highlight bash %}
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package postgresql-9.3
E: Couldn't find any package by regex 'postgresql-9.3'
{% endhighlight %}
Turned out Ubuntu 13.10 (saucy) which is on my desination server doesn't have 9.3 version. We made backup of 9.3 DB, so we need to restore it in 9.3. Here is workarround:

Create `pg.list` file in `/etc/apt/sources.list.d`. Add `deb http://apt.postgresql.org/pub/repos/apt/ saucy-pgdg main` line to it and make 

{% highlight bash %}
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install postgresql-9.3 postgresql-server-dev-9.3
{% endhighlight %}
Now let's restore DB at new place. Make sure postgresql is stopped

{% highlight bash %}
sudo tar -xf backup.tar -C /
{% endhighlight %}
Start postgre service. Now you should have your fully restored DB. Enjoy!