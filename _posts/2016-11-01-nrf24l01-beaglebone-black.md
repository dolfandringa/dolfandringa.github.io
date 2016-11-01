---
layout: post
title: NRF24L01 and SPIDEV on the Beagle Bone Black
category: blog
date: "2016-11-01"
tags: 
    - wireless
    - sensor_node
    - beaglebone
    - nrf24l01
    - linux
    - electronics
---
I am working on various electronics projects where I want to have a sensor node that collects data from a sensor and either logs it to a micro-SD-card directly or sens it wirelessly to another machine that uses/logs/etc the data. I opted for a set of [NRF24L01+](http://www.nordicsemi.com/eng/Products/2.4GHz-RF/nRF24L01) boards for the wireless communication between the nodes and data loggers. So to test it out I want to communicate between my [Beagle Bone Black](http://www.nordicsemi.com/eng/Products/2.4GHz-RF/nRF24L01) and an [Arduino](https://www.arduino.cc/). The [NRF24 library by TMRh20](https://www.arduino.cc/) supplies a unified API for the NRF24L01 that should work on Arduino, linux (like Beagle Bone Black and Raspberry Pi) and other platforms. It also includes some examples that are identical across platforms so you can test between platforms with the same code. Really cool. I had trouble getting it to work on my Beagle Bone Black. This post hopefully will help other getting it to work on a Beagle Bone Black running Debian Jessy.

I basically followed two howto's, the one on HTTPS://tmrh20.github.io/RF24/Linux.html and the one on HTTPS://electron14.com/?p=404. The latter of the two uses another library, and I want to use the TMRh20 to keep things uniform across platforms if possible, but still it was of help. The wire connections on that page are correct. And the TMRh20 library also installed fine if I followed that how-to. But as soon as I started the "gettingstarted" example from there, I got the error
{% highlight bash %}
RF24/examples/GettingStarted/
can't open device: No such file or directory
Aborted
{% endhighlight %}
It took me quite a while to figure out what was going wrong. 

The NRF24L01 uses the SPI bus for communication. The Beagle Bone Black uses SPIDEV for this and has two SPI buses. Those need to be enabled, and for them (or I believe actually only 1 of the 2) to work, the HDMI bus needs to be disabled. What I had to do was first edit `/bootuEnv.txt` and make sure the following two lines were in there:
{% highlight bash %}
cape_disable=bone_capemgr.disable_partno=BB-BONELT-HDMI,BB-BONELT-HDMIN
cape_enable=bone_capemgr.enable_partno=BB-SPIDEV0,BB-SPIDEV1
{% endhighlight %}
After that you need to restart your beaglebone. With
{% highlight bash %}
ls /dev/spi*
/dev/spidev1.0  /dev/spidev1.1  /dev/spidev2.0  /dev/spidev2.1
{% endhighlight %}
you can see the devices are there.
You can also see they are available with
{% highlight bash %}
# cat /sys/devices/platform/bone_capemgr/slots 
0: PF----  -1 
1: PF----  -1 
2: PF----  -1 
3: PF----  -1 
4: P-O-L-   0 Override Board Name,00A0,Override Manuf,BB-SPIDEV0
5: P-O-L-   1 Override Board Name,00A0,Override Manuf,BB-SPIDEV1
{% endhighlight %}

Still the RF24 library doesn't work though. This is because for some reason it uses the wrong device file.
During install of the library (see howto above) the code for me was downloaded in ~/rf24libs. In that folder I needed to change the file *RF24/utility/SPIDEV/spi.cpp*. On line 21 you need to change
{% highlight c %}
this->device = "/dev/spidev0.0";
{% endhighlight %}
to
{% highlight c %}
this->device = "/dev/spidev1.0";
{% endhighlight %}

After that you need to run
{% highlight bash %}
./configure
make
make install
{% endhighlight %}
in the *~/rf24libs/RF24* directory to re-compile and install the library. After that the gettingstarted examples should start.

