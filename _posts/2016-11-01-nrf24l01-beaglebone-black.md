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

For me still I couldn't get the library to work though. Turns out it is because of differences in the arguments between Linux and Arduino. With Arduino you give the radio() function the pin numbers of pins the CE and CSN pings of the board are connected to. With the BeagleBone BLack and SPIDEV the first argument is the pin number of the CE pin, while the second argument is the number of the SPI device that it is connected to. If you connected it as on the electron14.com tutorial, the CE pin is connected to pin GPIO_125 (pin 27 of header P9) and you're using SPI1.0. 
In order to specify these pins in the gettingstarted example, you need to make sure there is only one line in *~/rf24libs/RF24/examples_linux/gettingstarted.cpp* which instantiates the RF24 radio with the correct numbers:
{% highlight c %}
RF24 radio(125, 10);
{% endhighlight %}
THe first argument specifies the CE pin is GPIO_125 while the second says that it uses SPI1.0, or in other words */dev/spidev1.0*. 

After that you need to run
{% highlight bash %}
make gettingstarted
{% endhighlight %}
in the *~/rf24libs/RF24/examples_linux* directory to compile it.
After that, the example works:
{% highlight bash %}
# ./gettingstarted 
RF24/examples/GettingStarted/
STATUS       = 0xff RX_DR=1 TX_DS=1 MAX_RT=1 RX_P_NO=7 TX_FULL=1
RX_ADDR_P0-1     = 0xffffffffff 0xffffffffff
RX_ADDR_P2-5     = 0xff 0xff 0xff 0xff
TX_ADDR      = 0xffffffffff
RX_PW_P0-6   = 0xff 0xff 0xff 0xff 0xff 0xff
EN_AA        = 0xff
EN_RXADDR    = 0xff
RF_CH        = 0xff
RF_SETUP     = 0xff
CONFIG       = 0xff
DYNPD/FEATURE    = 0xff 0xff
Data Rate    = 1MBPS
Model        = nRF24L01
CRC Length   = 16 bits
PA Power     = PA_MAX

 ************ Role Setup ***********
 Choose a role: Enter 0 for pong_back, 1 for ping_out (CTRL+C to exit) 
 >
 {% endhighlight %}

