# Installing ka9q-radio

The ka9q-radio package is linux-only and has been installed on both Debian-based systems, like the Raspberry Pi, and Ubuntu on X86 hardware - and it can likely be installed on a wide variety of other hardware platforms.

## Hardware considerations

Network connectivity - wired Ethernet is strongly recommended!

The ka9q-radio package uses multicast extensively for communications and as such, it is not recommended that it be installed on systems that lack a physical Ethernet connection and are only capable of Wi-Fi.  The reason for this is that the traversal of Multicast across wireless networks is fraught with difficulties:  If you are installing ka9q-radio for the first time it is strongly recommended that you avoid the hassle of wireless connections of any sort until you get an understanding of how ka9q-radio works on a wired network.  In other words, you should not complicate your learning curve any more than is necessary.

Having said that, there are utilities within ka9q-radio that can be used to traverse networks for which multicast is not suited - specifically those using the Opus open source codec - but seriously, you should really avoid any sort of wireless connection until after you are thoroughly familiar with ka9q-radio, its connectivity requirements, and how these requirements may (or may not be) met by your local network.


Suitable USB interfacing and host computer processing capability

All of the SDR hardware currently supported by ka9q-radio is interfaced to the host computer via USB - and most of this hardware (RTL-SDR, AirSpy, HackRF, SDRPlay) will work well with the limitations imposed by the USB2 interface - that is, data rates on the order of 400-450 Mbits/sec and depending on the capability of the SDR hardware and the bit depth, this will allow up to around 10 MHz of spectrum to be covered.

Some hardware (e.g. the RX-888 and variants) are largely direct-sampling SDRs and their capabilities can well-exceed USB2 rates, requiring USB3 to unleash the full potential.  Testing indicates that an RX-888 Mk2 on a good-quality USB3 interface will allow a sample rate exceeding 120 MHz - but this is starting to get close to the practical limits of the USB3 protocol/hardware.

As for the choice of the processor on the host computer, in the Raspberry Pi world, a Raspberry Pi 4 or better is recommended as this has the capability of processing several MHz of spectrum at once.  If your chosen SDR has greater bandwidth than this, you should be looking toward a mid-range Intel i5 or i7 as these - coupled with a USB3 interface - will be capable of processing at least 65 megasamples/second with ka9q-radio.

The other system resources (drive space, free RAM) are minimally important in that the requirements of ka9q-radio are modest:  Far more of such resourced will be consumed by the operating system than by ka9q-radio itself.

CLI or GUI

To minimize resource utilization, a CLI-only (Command Line Interface) installation of Linux is more than adequate for use with ka9q-radio in its intended role:  As a processor of RF into receive streams.  If you with to use additional applications atop this (say, fldigi or other graphically-based programs) a GUI might be warranted - but keep in mind that if you are doing wideband processing of data (e.g. "listening" to the entirety of the HF spectrum with an RX-888) this, alone, will take significant CPU resources, require appropriate attention when it comes to task scheduling and prioritization to avoid drop-outs and other types of data loss.  In other words, if you are going to process a lot of spectrum, it's best to dedicate one computer to this task and do the "graphical stuff" on another.



## Getting ka9q-radio

The ka9q-radio package may be obtained from Github from:  https://github.com/ka9q/ka9q-radio

Typical installation:

Be sure to install ka9q-radio from the user under which you plan to run it and make sure that this user has the necessary privileges to access the hardware needed.  You will also need to make sure that git is installed on your computer.

From the appropriate directory - typically the root directory of the user to run ka9q-radio do:  sudo git clone https://github.com/ka9q/ka9q-radio

The result of this should be a successful cloning (download) and the creation of the directory "ka9q-radio".

At this point, take a look at the file in ka9q-radio called "INSTALL.txt" - link:  This has instructions related to building the binaries and installing ka9q-radio on your system and is the authoritative source for such information.  

In short:

sudo apt install build-essential libusb-1.0-0-dev libusb-dev libncurses5-dev libfftw3-dev libbsd-dev libhackrf-dev libopus-dev libairspy-dev libairspyhf-dev librtlsdr-dev libiniparser-dev libavahi-client-dev portaudio19-dev libopus-dev

Note that some of these dependencies are hardware-specific, including:
libhackrf-dev - HackRF
libairspy-dev and libairspyhf-dev - AirSpy
librtlsdr-dev - RTL-SDR Dongle
If you do not plan to use specific hardware, you may not need to install those pieces, but there's really no harm in installing everything.

NOTE:  If you plan to use an SDRPLay receiver (RSP1a, RSPdx, etc.) you should install the SDRPlay API BEFORE you build ka9q-radio - or be prepared to manually build the SDRPlay-related modules later.

After installing the above dependencies, do:
```
ln -s Makefile.linux Makefile 
```
If you are installing on a Raspberry Pi, do:
```
ln -s Makefile.pi Makefile
```


Then:

```
make
sudo make install
```

This will write into (and create as necessary) the following directories:

```
/usr/local/sbin               daemon binaries (e.g., 'radio')
/usr/local/bin                application programs (e.g., 'control')
/usr/local/share/ka9q-radio   support files (e.g., 'modes.conf')
/var/lib/ka9q-radio           application state files (e.g., tune-*)
/etc/systemd/system           systemd unit files (e.g., radio@.service)
/etc/sysctl.d                 system configuration files (e.g., 98-sockbuf.conf)
/etc/udev/rules.d             device daemon rule files (e.g., 52-airspy.rules)
/etc/fftw                     FFTW "wisdom" files (i.e., wisdomf)
/etc/radio                    program config files (e.g., radio@2m.conf)
```

Before you proceed:

Before you proceed it is recommended that you give appropriate permission for the FFT "Wisdom" file:  This file - used to optimize the Fast-Fourier Transformations that process the signal data - must be accessible to the user running ka9q-radio as follows:

After installing and building ka9q-radio, run the following commands (sudo may be required):

```
mkdir /var/lib/ka9q-radio                                   Note:  This may fail if it already exists
chown <username> /var/lib/ka9q-radio               Substitute the user name under which you are running "ka9q-radio"
```
!!! Inconsistencies in this text !!!

It may be worth verifying that /var/lib/ka9q-radio/wisdom is "owned" by the user running "ka9q-radio".

Also make sure that this directory - and the wisdom file - belong to the same group under which you are running "ka9q-radio" using "chgrp".  If, when starting "radiod" you see an error related to the wisdom file it probably has to do with access to it.

At this point ka9q-radio should be installed.


IMPORTANT - Back up your .conf files!

As of the time of this writing (June, 2023) the default configuration files WILL BE OVERWRITTEN every time you do an update/make of ka9q-radio.  Both the files in the home "~/ka9q-radio" and "/etc/radio" directories can be overwritten.

What this means is that if you modify the original configuration files (e.g "/etc/radio/rx888d.conf", "/etc/radio/radiod@hf.conf" - and those in the "ka9q-radio directory) you will LOSE those modifications when you do an update.

When you make changes to ANY configuration file, be sure to save a copy elsewhere, and be prepared to restore it after you do updates.

It is on the list of future updates to change this behavior.




FFT optimization

As noted earlier, ka9q-radio makes extensive use of the FFTW3 algorithm for processing signals.  This algorithm is, by far, the most CPU-intensive portion of ka9q-radio and to reduce processor loading, it can be optimized for the current computing environment.  This optimization is computer-specific and should be done for each, individual installation of ka9q-radio.

Note:  This optimization is not required, but it can save significant processor overhead which can make the difference between usability and not on a system with limited resources.  If you are just playing around or have CPU power to burn, you can get away without doing this if you like.

The document describing the the task of FFT optimization is found in:  /ka9q-radio/docs/FFTW3.md (link).  Aspects of that file are included below for convenience but the reader should refer to that file as the authoritative source and read it.

From the ka9q-directory do:   

```
time fftwf-wisdom -v -T 1 -o nwisdom rof500000 cof36480 cob1920 cob1200 cob960 cob800 cob600 cob480 cob320 cob300 cob200 cob160
```

This program may take some time to run - from just a few minutes on a fairly fast computer to several hours on a much slower computer, so be patient!

When done, a file called "nwisdom" will be placed in the ka9q-radio directory.  Go to the directory /etc/fftw and back up the file "wisdomf" (e.g. rename it to something else) and then copy the newly-created nwisdom file there, naming it as "wisdomf", making sure that it has the same permissions/user/group as the original "wisdomf" file.

!!! Inconsistencies in this text !!!

`# References


**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico. Video [Phil Karn at TAPR DCC 2018](https://youtu.be/D1LYLDGknOY)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://www.youtube.com/watch?v=VrMoNnctrqo&t=13s). 40th ARRL and TAPR Digital Communications Conference, [Virtual On-Line](https://youtu.be/kVY3E3e--_I?t=15080)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2022.  SDR Forum Dayton Hamvention. [PDF](https://files.tapr.org/meetings/DCC_2018/DCC2018-KA9Q-Multicast4SDR-Interconnect.pdf)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2023.  [PAPA club meeting 2023](https://youtu.be/7nhBFSGby2o)
