---
layout: post
title: NRF24L01 and SPIDEV on the Beagle Bone Black
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

There are various howto's about the Beagle Bone Black and RF24, and as so often with the BBB, there are many articles contradicting eachother, using different pinouts than my BBB uses I guess. What I have found is that the only reliable pinout for a BBB is the one specified on the standard webinterface of your BBB (on the front-page at the bottom). So the only really useful how-to I ended up using is about the RF24 library on [https://tmrh20.github.io/RF24/Linux.html], which doesn't talk about the BBB specifically. So after installing the RF24 library I just tried what some other how-to's said but as soon as I started the "gettingstarted" example from there, I got the error
{% highlight bash %}
RF24/examples/GettingStarted/
can't open device: No such file or directory
Aborted
{% endhighlight %}
It took me quite a while to figure out what was going wrong. It turned out to be various problems, mostly related to the SPI bus and it's pinout.

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

BB-SPIDEV0 I think corresponds to /dev/spidev1\* and BB-SPIDEV1 to /dev/spidev2\*. in the filesystem. If you look at the pinout for my Beagle Bone Black (but be sure to check yours on the web-interface of yours) specifies the following pinouts:

* ![Beagle Bone Black SPI pinout](/images/bbb_spi_pinout.png).

There, the pins SPI1_DO, SPI1_D1, SPI1_CSO and SPI1_SCLK correspond to BB-SPIDEV1 and /dev/spidev2\*. So I wired my NRF24L01+ in the following way:

| Beagle Bone| NRF pin name | NRF pin number |
|------------|--------------|----------------|
| DGND       | GND          | 1              |
| VDD3V3     | VCC          | 2              |
| GPIO_115   | CE           | 3              |
| SPI1_CSO   | CSN          | 4              |
| SPI1_D0    | MOSI         | 6              |
| SPI1_D1    | MISO         | 7              |
| SPI1_SCLK  | SCLK         | 5              |


After having it wired like this I wanted to get the gettingstarted sample in *~/rf24libs/RF24/examples_linux/gettingstarted.cpp* to work. The problem I had there is that the RF24 radio() statement in there, which instantiates the RF24 radio is different on Linux than it is on Arduino. On Arduino you give the radio() function the pin numbers to which the CE and CSN pins of the NRF24 are connected. With the Beagle Bone Black and SPIDEV the first argument is the pin number of the CE pin, while the second argument is the number of the SPI device that it is connected to. If you connected it as I showed above, the CE pin is connected to pin GPIO_115 (pin 27 of header P9) and you're using SPI1. So the line with RF12 radio() in *~/rf24libs/RF24/examples_linux/gettingstarted.cpp* should look like this:
{% highlight c %}
RF24 radio(115, 20);
{% endhighlight %}
The first argument specifies the CE pin is GPIO_115 while the second says that it uses SPI1, or in other words */dev/spidev2.0*. 

After that you need to run
{% highlight bash %}
make gettingstarted
{% endhighlight %}
in the *~/rf24libs/RF24/examples_linux* directory to compile it.
After that, the example works (provided you have another device, like an Arduino, with an NRF24L01 listening to pong back):
{% highlight bash %}
# ./gettingstarted
RF24/examples/GettingStarted/
STATUS       = 0x0e RX_DR=0 TX_DS=0 MAX_RT=0 RX_P_NO=7 TX_FULL=0
RX_ADDR_P0-1     = 0x65646f4e34 0x65646f4e33
RX_ADDR_P2-5     = 0xc3 0xc4 0xc5 0xc6
TX_ADDR      = 0x65646f4e34
RX_PW_P0-6   = 0x20 0x20 0x00 0x00 0x00 0x00
EN_AA        = 0x3f
EN_RXADDR    = 0x02
RF_CH        = 0x4c
RF_SETUP     = 0x03
CONFIG       = 0x0e
DYNPD/FEATURE    = 0x00 0x00
Data Rate    = 1MBPS
Model        = nRF24L01+
CRC Length   = 16 bits
PA Power     = PA_LOW

 ************ Role Setup ***********
 Choose a role: Enter 0 for pong_back, 1 for ping_out (CTRL+C to exit) 
 >1
 Role: Ping Out, starting transmission 

 Now sending...
 Got response 525939321, round-trip delay: 5
 Now sending...
 Got response 525940329, round-trip delay: 2
 Now sending...
 Got response 525941333, round-trip delay: 2
 Now sending...
 Got response 525942337, round-trip delay: 2
{% endhighlight %}

