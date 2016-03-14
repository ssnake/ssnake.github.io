---
layout: post
title: Simple setup Raspberry Pi as Webcam
comments: true
tag: rasp, camera
---

I'm owner of 256mb version Raspberry PI. Recently I've bought a [camera module](http://www.raspberrypi.org/camera-board-available-for-sale/). Here is my tutorial how to make a web camera. 

The idea is simple. We will capture images every 5 seconds and will share them via a web interface. For capturing we will use the `raspistill` console utility which is provided with RASPBIAN as out-of-box. 

### Setup RAM drive for storing captured images
Since any SD card has limit resources for writing cycles we will use ram drive as temporary storage drive. Let's create a ram drive. First make a folder for it:

{% highlight console %} 
sudo mkdir /var/tmp/ram_drive

{% endhighlight %}

Add following string to fstab to make sure a ram drive is created on an every system boot

{% highlight console %}
tmpfs /var/tmp/ram_drive tmpfs nodev,nosuid,size=10M 0 0 
{% endhighlight %}

Now mount it to a system:

{% highlight console %}
sudo mount -a
{% endhighlight %}

Check if it's mounted:

{% highlight console %}
df -h
{% endhighlight %}

### Create bash script to capture images

We will create a script which will be launched by the cron daemon every minute. 1 minute is not enough we need refresh image every 5 secs. The solution is run `raspistill` 12 times as one cron task (60/12 = 5 sec). Furthermore this script will work with 2 image files. One image for publishing it via web daemon, another is kind of "double buffer" file. Also timestamp will be added to images.
Install `imagemagick` which is responsable for this work.

{% highlight console %}
sudo apt-get install imagemagick
{% endhighlight %}

Open nano editor and copy/past the script:

{% highlight bash %}
#!/bin/bash
OUTPUT=/var/tmp/ram_drive
OPTIONS='-vf -hf -w 800 -h 600 -q 80 -x'
#OPTIONS='-vf -hf -w 1024 -h 768 -q 80 -x'
DATE=$(date +"%d/%m/%Y")
HOUR=$(date +"%R")

for i in `seq 1 $1` 
do
  echo "pic $i"
  raspistill -o $OUTPUT/buf.jpg $OPTIONS

  convert $OUTPUT/buf.jpg \
   -pointsize 14 -fill white -annotate +670+590  \
   $DATE \
   -pointsize 14 -fill white -annotate +750+590  \
   $HOUR \
  $OUTPUT/pre_cam.jpg


  #move result file to master file
  mv $OUTPUT/pre_cam.jpg $OUTPUT/cam.jpg
  if [[ $i != $1 && "$1" != "" ]]
  then
    echo "wait"
    sleep 5
  else
    echo "do not wait"
  fi
done


{% endhighlight %}

Make it executable:

`chmod +x cam.sh`

### Install web daemon

We will use web to have access to our camera. Since rasp pi is small device with lack of resources we need to make sure we use best software which give us best performance. According to this [comparison](https://www.jeremymorgan.com/blog/programming/raspberry-pi-web-server-comparison/) the best web daemon  is [Monkey](http://monkey-project.com/raspberry). Let's install it.

Add source repo to `/etc/apt/sources.list` file:

{% highlight console %}
deb http://packages.monkey-project.com/primates_pi primates_pi main
{% endhighlight %}

Now install and config Monkey:

{% highlight console %}
sudo apt-get update &&  sudo apt-get install monkey
{% endhighlight %}

After installation the monkey daemon should be up. Open browser and type `<your pi address>:2001` you should see default monkey welcome page. The default place for web site is in `/usr/share/monkey`. Go there and remove everything. Now make sym link to ram drive image.

{% highlight console %}
cd /usr/share/monkey
sudo ln -s /var/tmp/ram_drive/cam.jpg cam.jpg
{% endhighlight %}
By default monkey doesn't support sym links you have to enable it. Open `/etc/monkey/monkey.conf` and enable sym links:

{% highlight console %}

SymLink On
{% endhighlight %}
Restart daemon:

{% highlight console %}
sudo service monkey restart
{% endhighlight %}

### Web page

Let's create web page which will refresh image every 5 secs.

index.html

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="/main.css">
    <script src="script.js"></script>
  </head>
  <body>

  <img id="cam" class="cam" src="/cam.jpg">
  <body>
</html>
{% endhighlight %}
main.css

{% highlight css %}
.cam {
  height: 600px;
  width: 100%;
}

{% endhighlight %}

script.js

{% highlight js %}
window.onload = function() {
  setInterval(function() {
    var myImageElement = document.getElementById('cam');
    myImageElement.src = 'cam.jpg?rand=' + Math.random();
  }, 5000);
}

{% endhighlight %}
Check `<your pi address>:2001` you should see updatable pictures from pi.

### Conclusion

This is not a best solution for "streaming" pi's pictures online. I came across better solutions on internet.  However, this article has helpful tricks I may need in future.
