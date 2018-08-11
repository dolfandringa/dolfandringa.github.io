---
layout: post
title: Problems getting MPD, PulseAudio and Arch Linux to work together
date: "2016-02-20"
tags: 
    - ArchLinux
    - MPD
    - PulseAudio
---
I love Gnome Music Player Client (gmpc) as an audio player, with MPD (Music Player Daemon) as a backend. Both because of the remote control possibilities but GMPC is also just a great client. But when installing MPD, combined with PulseAudio on my Arch Linux laptop, I ran into trouble. MPD runs as a different user, and as that user wasn't able to connect to PulseAudio that runs under my regular desktop user.

It turns out that you can run PulseAudio in a few different ways, but on my Mate desktop in Arch Linux, PulseAudio is started with my desktop session when I login. The audio programs I run on my desktop connect to this PulseAudio session and work fine. The MPD daemon runs as a different user and can't connect to the PulseAudio session running as my desktop user. The trick is to spawn a PulseAudio session for each user individually. [According to the MPD wiki](http://mpd.wikia.com/wiki/PulseAudio) most distributions have pulse audio setup to auto-spawn a PulseAudio session for each user. On my Arch Linux box that wasn't the case though. This resulted in the following error messages:

{% highlight bash %}
pulse_output: Failed to enable "PulseAudio Analog Out" [pulse]: pa_context_connect() has failed: Connection refused
output: Failed to open audio output
pulse_output: Failed to open "PulseAudio Analog Out" [pulse]: failed to connect: Access denied
{% endhighlight %}

The solution is to edit /etc/pulse/client.conf and set ```autospawn = yes``` and make sure the line is not commented with a ```;``` at the beginning.
After this, restart mpd and pulseaudio should work. To check you can run ```ps auwx|grep pulse```. It should show output similar to the following:

{% highlight bash %}
<myuser> 11378  0.0  0.1 497404 11512 ?        S<sl 08:06   0:00 /usr/bin/pulseaudio --daemonize=no
mpd      11513  0.3  0.1 563304 12604 ?        Sl   08:14   0:04 /usr/bin/pulseaudio --start --log-target=syslog
{% endhighlight %}

You see a pulseaudio session for your own user and one for the mpd user. If this is the case, you should be able to play music with MPD to pulseaudio.
