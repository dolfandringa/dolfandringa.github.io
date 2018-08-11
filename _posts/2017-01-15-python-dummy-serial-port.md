---
layout: post
title: Creating dummy serial ports in pure python.
date: "2017-01-15"
tags: 
    - python
    - pyserial
    - solar
    - pseudoterminal
---
Recently I needed to reverse-engineer the serial communication between a solar inverter and a computer. Although hardware manufactures may often be good at making the hardware they sell, their accompanying software often leaves a lot to be desired. To develop my own software, I needed to interface with a specific inverter, both to intercept the serial communication at first, but later to write my own software. In order to be able to write unittests for the serial port, and to test my software without actually being connected to an inverter, I wanted to have a dummy serial port on my computer that listens for any connections to the port and responds whatever I tell it to.

Many [questions have been asked on stackoverflow](http://stackoverflow.com/questions/15173614/how-can-i-use-a-pseudoterminal-in-python-to-emulate-a-serial-port) about this (including by me), and an often recurring answer seems to be using software like socat for this. Socat definitely works well for communicating with serial ports, dumping data from serial ports to files, etc. I needed to also write tests for a piece of software that I was writing in python. Again, Mock protocols can be used for this and are often the best solution, but I was stubborn and wanted to figure out how to setup a dummy serial port that other software can communicate with.

The trick is to use [pseudoterminals](https://en.wikipedia.org/wiki/Pseudoterminal) in Linux. Pseudoterminals can be used (and are used) for many things, but serial communication can be one. Basically it is just a file that software can write to or read from as if it was a serial connection. Pseudoterminals work with a master and slave. One end is the master, which in my case is where my "dummy serial device" is listening. The slave side is the code/software I wanted to test. The master is usually /dev/ptmx on linux, while the slave is often of the form /dev/pts/&lt;x&gt; where x is any integer. The master endpoint is constant, even with multiple psuedoterminal connections, the master endpoint doesn't change. The slave endpoints do have one for each psuedoterminal.

This is where the trick was that I ran in to. Python has a [pty module](https://docs.python.org/3/library/pty.html) that you can use to setup this master and slave pair. The [pty.openpty()](https://docs.python.org/3/library/pty.html#pty.openpty) function returns two file descriptors, one for the master and one for the slave. These file descriptors already are opened and can be read from with [os.read](https://docs.python.org/3/library/os.html#os.read) or written to with [os.write](os.write). They are not filenames but file descriptors. With [os.ttyname](https://docs.python.org/3/library/os.html#os.ttyname) you can retrieve the actual filename of those file descriptors. But if you create a new file object for the master endpoint, you actually create a new file descriptor and instead of using the already opened connection to the slave, a new pseudoterminal (slave) will be created, with a new connection to the master. So while you can re-open the slave multiple times with different file descriptors and keep using the same connection to the master, you can't open multiple file descriptors to the master that connect to the same slave.

So to show some code with comments:
{% highlight python %}
import os, pty
from serial import Serial
import threading

def listener(port):
    #continuously listen to commands on the master device
    while 1:
        res = b""
        while not res.endswith(b"\r\n"):
            #keep reading one byte at a time until we have a full line
            res += os.read(port, 1)
        print("command: %s" % res)

        #write back the response
        if res == b'QPGS\r\n':
            os.write(port, b"correct result\r\n")
        else:
            os.write(port, b"I dont understand\r\n")

def test_serial():
    """Start the testing"""
    master,slave = pty.openpty() #open the pseudoterminal
    s_name = os.ttyname(slave) #translate the slave fd to a filename

    #create a separate thread that listens on the master device for commands
    thread = threading.Thread(target=listener, args=[master])
    thread.start()

    #open a pySerial connection to the slave
    ser = Serial(s_name, 2400, timeout=1)
    ser.write(b'test2\r\n') #write the first command
    res = b""
    while not res.endswith(b'\r\n'):
        #read the response
        res +=ser.read()
    print("result: %s" % res)
    ser.write(b'QPGS\r\n') #write a second command
    res = b""
    while not res.endswith(b'\r\n'):
        #read the response
        res +=ser.read()
    print("result: %s" % res)

if __name__=='__main__':
    test_serial()
{% endhighlight %}

This script has a listener function in a separate thread constantly listening on the master side for commands and responding accordingly. The test_serial function first starts the listener thread and then uses [pySerial](https://pythonhosted.org/pyserial/) to open a serial connection to the slave. It sends a command and reads the response. And then repeats it with a different command.

I hope this clarifies how to use psuedoterminals for dummy serial connections in python.
