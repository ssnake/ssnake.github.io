---
layout: post
title: SSL in Rails
comments: true
---

A few words about terms. You may be aware of SSL and TLS. According to wiki SSL 3.0 is predecessor of TLS 1.0. It may be allowed to say that TLS is SSL 3.1. US Government has already prohibited using of SSL in their sensitive communications. SSL is old and have a lot of vulnerabilities. It's believed that TLS is more secure than SSL. Ok, let's get down to work..


## Key Generation

`openssl genrsa -aes128 -out fd.key 2048`

* genrsa - generate a rsa key
* -aes128 - the key will be protected byt AES-128
* -out fd.key - the name of output file name of key
* 2048 - size of the key


## Creating Certificate Signing Requests

With having the key now let's create CSR file. This file will have all sensitive information

`openssl req -new -key fd.key -out fd.csr`


##Signing Your Own Certificates

Now you can sign your CSR file:

`openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt`

If you don't want to create CSR file as a single step use following command:

`openssl req -new -x509 -days 365 -key fd.key -out fd.crt`

Answer all questions as you wish, but when it asks `Common Name (e.g. server FQDN or YOUR name)` input the name that you point out in apache <your-site>.conf as a ServerName. Let's it be `www.myapp.com`.

## Apache Configuration

I'm using for my apps apache as a web server. Here are steps to setup it to use SSL. First of all we need to disable a key password. 

{% endhighlight %}
openssl rsa -in fd.key -out fd_np.key
mv fd.key fd.key.org
mv fd_np.key fd.key

{% endhighlight %}

Check if we get rid of a password: `openssl rsa -text -in fd.key`.

Now copy `fd.key` and `fd.crt` to Ubuntu Trust Store


{% endhighlight %}
sudo cp fd.crt /etc/ssl/certs/
sudo cp fd.key /etc/ssl/private/
sudo chmod 0600 /etc/ssl/private/fd.key
{% endhighlight %}

For using SSL apache has a module for this. Let's enable it:

{% endhighlight %}
sudo a2enmod ssl
{% endhighlight %}

Now we have to re-write our <site>.conf in `/etc/apache2/sites-enabled`

{% endhighlight %}
   # change default port to 443
   <VirtualHost *:443>                                                                                                                          
      ServerName www.myapp.com                                                                                                                                                                                                  
      DocumentRoot /var/www/myapp/current/public                                                                                          
      <Directory /var/www/myapp/current/public>                                                                                           
         # This relaxes Apache security settings.                                                                                              
         AllowOverride all                                                                                                                     
         # MultiViews must be turned off.                                                                                                      
         Options -MultiViews                                                                                                                   
         # Uncomment this if you're on Apache >= 2.4:                                                                                          
         Require all granted   



      </Directory>              
        # Add SSL stuff here
  		SSLEngine on
  		SSLCertificateFile    /etc/ssl/certs/fd.crt
		SSLCertificateKeyFile /etc/ssl/private/fd.key
   </VirtualHost> 
 ```
 And now final step

 ```
 sudo service apache2 restart
 ```

 ##Rails

 if you want to run your RoR app in ssl mode add ` force_ssl` to your application controller.

 If you're using `devise` force it to use ssl as well. Add these lines to `config/environments/production.rb`

 ```
config.to_prepare { Devise::SessionsController.force_ssl }
config.to_prepare { Devise::RegistrationsController.force_ssl }
config.to_prepare { Devise::PasswordsController.force_ssl }



