---
layout: post
title:  "Control your TV with a Raspberry Pi"
date:   2016-6-7
categories: projects
---

In this tutorial we'll be setting up a Raspberry Pi for use as an infrared remote control. If you use your Raspberry Pi for watching videos there might be better ways for controlling a TV, like HDMI-CEC if your TV supports it.

For this project you will need:

- Raspberry Pi
- Breadboard
- 1x IR LED
- 1x IR Receiver
- 1x 10k Ohm Resistor
- Jumper Wires

To control the TV from the Raspi, we'll use LIRC (the Linux Infrared Remote Control package). Check the [LIRC remotes database](http://lirc-remotes.sourceforge.net/remotes-table.html) to see if someone's already contributed the codes for your TV. If it's in there, you're in luck! And you don't have to buy an IR Receiver.

##### 1. Setup

Install LIRC:

{% highlight bash %}
$ sudo apt-get update
$ sudo apt-get install lirc
{% endhighlight %}

Fork the project files so you'll have a basic web remote:

{% highlight bash %}
$ cd ~
$ git clone https://github.com/fbradley8/raspi-tv-remote
{% endhighlight %}

Then download the required packages:

{% highlight bash %}
$ cd raspi-tv-remote
$ npm install
{% endhighlight %}

##### 2. Configuration

Update the LIRC configuration file to use your infrared receiver:

{% highlight bash %}
$ sudo nano /etc/lirc/hardware.conf
{% endhighlight %}

Make the following change:

{% highlight bash %}
DRIVER="default"
DEVICE="/dev/lirc0"
MODULES="lirc_rpi"
{% endhighlight %}

Update the boot config:

{% highlight bash %}
$ sudo nano /boot/config.txt
{% endhighlight %}

Add the following line:

{% highlight bash %}
dtoverlay=lirc-rpi,gpio_in_pin=17,gpio_out_pin=18
{% endhighlight %}

To make these changes we need to reboot the pi, but the next step is wiring so we might as well shut it down for now.

##### 3. Wiring

Using the required components, copy this wiring diagram.

![figure 1](/img/raspi-ir-remote-fig1.png)

If your remote wasn't listed in the [LIRC remotes database](http://lirc-remotes.sourceforge.net/remotes-table.html), you'll also need to wire up the IR receiver.

![figure 2](/img/raspi-ir-remote-fig2.png)

##### 4. Programming

Boot the pi back up and run:

{% highlight bash %}
$ irrecord -d /dev/lirc0 ~/lircd.conf
{% endhighlight %}

Program in as many keys as you like and then move the config file and restart the service.

{% highlight bash %}
$ sudo mv /etc/lirc/lircd.conf /etc/lirc/lircd_old.conf
$ sudo mv ~/lircd.conf /etc/lirc/
$ sudo /etc/init.d/lirc restart
{% endhighlight %}

##### 5. Test

If all is working fine, you should now be able to successfully run the following (assuming you called the power button key_power):

{% highlight bash %}
$ irsend SEND_ONCE Element key_power
{% endhighlight %}

##### 6. Setup your web-remote

Start the web server to view your webapp:

{% highlight bash %}
$ cd ~/raspi-tv-remote
$ npm start
{% endhighlight %}

Now browse to your pi's IP on port 8080 ex: http://x.x.x.x:8080

If you programmed your remote using the same namespacing as mine, it should work! If not don't worry. All you have to do is modify the id's of the keys in raspi-tv-remote/public/index.html.

The power button, for instance, has the ID attribute "key_power". If you set the power button as something else, replace this ID with that.

##### Wrap-up

Hopefully this tutorial got you up and running with a useable raspi-powered IR remote. The project files are fairly straightforward and you should easily be able to add other keys to the web interface.
