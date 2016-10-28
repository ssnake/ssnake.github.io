---
layout: post
title: Telegram gem and self-signed certificate
comments: true
---


If anyone faced an issue after setting [webhooks](https://core.telegram.org/bots/api#setwebhook) with self-signed certificate updates do not come to server. This article may help you.

For working with Telegram Bot I use  [telegram-bot-ruby gem](https://github.com/atipugin/telegram-bot-ruby)

Here is my rake tasks:

{% highlight ruby %}
desc 'create ssl' 
  task create_ssl: :environment do
  	cmd = "openssl req \
  -newkey rsa:2048 \
  -sha256 \
  -nodes \
  -keyout %{file_name_key} \
  -x509 \
  -days 365 \
  -out %{file_name_pem} \
  -subj \"/C=%{C}/ST=%{ST}/L=%{L}/O=%{O}/CN=%{host}\"
  " % 
   { 
        :file_name_key => 'telegram.key', 
        :file_name_pem => 'telegram.pem', 
        :C => "IT",
        :ST => "state",
        :L => "location",
        :O => "description",
        :host =>  ENV["CERT_HOST"] 
      }
      exec cmd
  end
{% endhighlight %}

{% highlight ruby %}
 desc "set webhook"
  task set_webhook: :environment do
  	Telegram::Bot::Client.run(TELEGRAM_TOKEN) do |bot|
		hook_url = ENV["BOT_HOST"] + "/webhooks/telegram"
		puts "Setting webhook url: #{hook_url}"
		resp = bot.api.set_webhook url: hook_url, certificate: File.open('telegram.pem')
		puts resp.inspect
  	end
  end
{% endhighlight %}



After creation self-signed certificate and then setting webhook  I ended up in a situation when I didn't get update from telegram.
I looked at nginx logs and saw following lines:

{% highlight logs %}
SSL_do_handshake() failed (SSL: error:14094418:SSL routines:ssl3_read_bytes:tlsv1 alert unknown ca:SSL alert number 48) while SSL handshaking
{% endhighlight %}

After days of googling I managed to figure out. Turned out 
{% highlight ruby %}
bot.api.set_webhook url: hook_url, certificate: File.open('telegram.pem')
{% endhighlight %}
 didn't do a work. It didn't upload certificate. 

Solution
=====

I used a curl command:
{% highlight bash %}
curl -F "url=https://<HOSTNAME>/webhooks/telegram" -F "certificate=@<FULL_PATH_TO_CERT.pem" https://api.telegram.org/bot<BOT_TOKEN>/setWebhook
{% endhighlight %}

After this everything started working properly.