---
layout: post
title: How to setup rails, puma and nginx
comments: true
tag: rails, production, puma, nginx, ubuntu
---

Let's suppose we have VPS server with ubuntu installed, we have developed a rails application that is ready to be deployed.
We want to run the app in production mode.

At the end of this tutorial we will have the app that uses puma and revealed to the world via nginx.

Let's start!

Make sure you have installed following packages on VPS:

{% highlight bash %}
sudo apt-get install gcc g++ make libssl-dev sqlite3 libsqlite3-dev libpq-dev postgresql
{% endhighlight %}


Open `Gemfile` and add following gems:
{% highlight ruby %}
group :development do
  gem 'capistrano', '~> 3.8'
  gem 'capistrano-lets-encrypt'
  gem 'capistrano-rbenv', '~> 2.0'
  gem 'capistrano3-puma'
  gem 'capistrano-rails'
  gem 'sshkit-sudo'
end
{% endhighlight %}

Generate capistrano files:

{% highlight bash %}
$ bundle exec cap install
mkdir -p config/deploy
create config/deploy.rb
create config/deploy/staging.rb
create config/deploy/production.rb
mkdir -p lib/capistrano/tasks
create Capfile
Capified
{% endhighlight %}

Uncomment and add these lines in `Capfile`:
{% highlight ruby %}
require "capistrano/rbenv"
require "capistrano/rails/assets"
require "capistrano/rails/migrations"
require "capistrano/bundler"
require 'capistrano/puma'
require 'capistrano/puma/jungle'
require 'capistrano/puma/workers'
require 'capistrano/puma/nginx'
require 'capistrano/lets-encrypt'
require 'sshkit/sudo'
{% endhighlight %}

Now trarget capistrano to your VPS server with editing of `config\deploy\production.rb`:

{% highlight ruby %}
server 'www.my_server.com', user: 'my_vps_login', roles: %w[app db web], primary: true
set :lets_encrypt_domains, 'www.my_server.com'
{% endhighlight %}

Nice. Now we need to edit `config\deploy.rb` file. Please add these lines:

{% highlight ruby %}

set :application, "my_app"
set :repo_url, "git@github.com:login/my_app.git"
set :branch, :master

set :deploy_to, '/home/my_vps_login/projects/my_app'
append :linked_files, "puma.conf"
append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/puma', 'tmp/sockets', 'public/system', 'certs'

set :rails_env, 'production'

set :puma_state, "#{shared_path}/tmp/puma/state"
set :puma_pid, "#{shared_path}/tmp/puma/pid"
set :puma_preload_app, true
set :puma_conf, "#{shared_path}/puma.conf"

set :lets_encrypt_roles, :web
set :lets_encrypt_user, 'my_vps_login'
set :lets_encrypt_email, 'my_email'
set :lets_encrypt_account_key, "~/#{fetch(:lets_encrypt_email)}.account_key.pem"
set :lets_encrypt_challenge_public_path, "#{release_path}/public"
set :lets_encrypt_output_path, '/etc/nginx/ssl'
set :lets_encrypt_local_output_path, '~/certs'
set :lets_encrypt_days_valid, 90

{% endhighlight %}

Ok, let's try to deploy the app:

{% highlight bash %}
$ bundle exec cap production deploy
...
ERROR linked file ~/projects/my_app/shared/puma.conf does not exist on www.my_server.com
...
{% endhighlight %}

You have to get errors. Let me explain a schema we're going to use.

Normally if you use Rails 5 it uses `puma` as default server.  When you start an app with `rails s` command `puma` reads `config\puma.rb` configuration file and gets started.
Since we're going to use `puma` on VPS things could be a bit different and we need different configuration file.
Thus in order to not mess with default configuration file we will use `puma.conf`. This file will be used only on VPS.

Run `bundle exec cap production puma:config` command it will create and uploade `puma.conf` to shared folder.

Or you can log on you VPS and create `~/projects/my_app/shared/puma.conf` and copy this:


{% highlight ruby %}

directory '/home/my_vps_login/projects/my_app/current'
rackup '/home/my_vps_login/projects/my_app/current/config.ru'

threads_count = ENV.fetch('RAILS_MAX_THREADS') { 5 }.to_i
threads threads_count, threads_count

environment ENV.fetch('RAILS_ENV') { 'production' }

prune_bundler
preload_app!

plugin :tmp_restart

pidfile '/home/my_vps_login/projects/my_app/shared/tmp/puma/pid'
state_path '/home/my_vps_login/projects/my_app/shared/tmp/puma/state'
bind 'unix:///home/my_vps_login/projects/my_app/shared/tmp/sockets/puma.sock'
bind 'tcp://127.0.0.1:3000'

activate_control_app

{% endhighlight %}


Try again. You should get something alike below:


{% highlight bash %}
$ bundle exec cap production deploy
...
 puma:start
      using conf file ~/projects/my_app/shared/puma.conf
      01 $HOME/.rbenv/bin/rbenv exec bundle exec puma -C ~/projects/my_app/shared/puma.conf --daemon
      01 Puma starting in single mode...
      01
      01 * Version 3.9.1 (ruby 2.4.1-p111), codename: Private Caller
      01
      01 * Min threads: 5, max threads: 5
      01
      01 * Environment: production
      01
      01 * Daemonizing...
...
{% endhighlight %}

You can control your puma server with commands:

{% highlight bash %}

$ bundle exec cap production puma:status
$ bundle exec cap production puma:start
$ bundle exec cap production puma:stop
$ bundle exec cap production puma:restart

{% endhighlight %}
Now we need to care about that our app is get started on system boot. For this we use jungle.
Do not use `cap production puma:jungle:install` instead  log in on VPS server. Download two bash scripts:

{% highlight bash %}
wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/init.d/puma
wget https://raw.githubusercontent.com/puma/puma/master/tools/jungle/init.d/run-puma
{% endhighlight %}

And follow this [instruction](https://github.com/puma/puma/tree/master/tools/jungle/init.d).

Now start out app

{% highlight bash %}
sudo /etc/init.d/puma start
{% endhighlight %}

Now is turn of nginx. It's easy. Just run:

{% highlight bash %}
bundle exec cap production puma:nginx_config
{% endhighlight %}

You will get `/etc/nginx/sites-available/my_app_production` conf file. Edit it as you needed.

Now restart nginx

{% highlight bash %}
sudo /etc/init.d/nginx restart
{% endhighlight %}

Your app should be up and ready. Enjoy!
