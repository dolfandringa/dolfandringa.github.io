---
layout: post
title: Remote PulseAudio volume change script for a guest-session user
category: blog
date: "2016-03-27"
tags: 
    - LinuxMint
    - guest-session
    - PulseAudio
    - ssh
    - command-line
    - script
---
I live and have setup the IT infrastructure for an NGO where I share my living space with volunteers. Generally the volunteers are hard workers and well behaved, but once in a while on a Saturday night they have a good time, which often involves turning the music up just a little too high. As I am usually already in my own little house a few 10's of meters away, I don't always share their enjoyment. But I don't want to nag them and am also way too lazy to go out and turn the music down. As the computer they use to play the music on runs Linux Mint and I can access it from my home through the wifi, I thought I'd create a script that allows me to change the volume on the computer remotely.

The problem is that they login using a guest-session, so every time they login, they get a different username and ID. And PulseAudio runs as that user, so if I change the volume as my own user I get a permission denied error. The following script first determines the username and uid of the user that they login as (line 1-3). Next (line 4-8) it determines the volume change direction from a command-line argument (if up is specified the volume goes up by 10% else down by 10%).
Then (line 11) it actually runs the command as the guest user. One thing you seem to need to do is set the XDG_RUNTIME_DIR environment variable for the user, which PulseAudio seems to use for something. If I run this script (or the alias I specified in my .bashrc) with sudo, it changes the volume correctly.
No more sleepless nights.

{% highlight bash linenos=table %}
#!/bin/bash
username=`who|grep "^guest[a-zA-Z0-9\-]\+" -o`
uid=`id -u $username`
if [[ $1 == 'up' ]];then
    dir='10%+'
else
    dir='10%-'
fi
su - $username -c "XDG_RUNTIME_DIR=/run/user/$uid amixer -D pulse sset Master $dir"
{% endhighlight %}
