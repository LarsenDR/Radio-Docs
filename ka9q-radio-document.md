
# Overview and Module documentation of ka9q-radio

by Dave Larsen, KV0S

---

# Overview

Phil Karn ka9q, developed a set of software defined radio receiver modules to demonstrate the utility of standard Internet real-time and multicast protocols for interprocess communication. The multicast streams convey complex digital IF sample streams, uncompressed and compressed audio and decoded physical layer frames (e.g., AX.25). One module can feed any number of others, and it is easy to move and restart individual modules without restarting the entire system. The package is currently being used for general analog reception and to process balloon APRS tracking transmissions but has many other applications including satellite operations and digital voice. The source code is in C, is available as open source 1 and runs on any UNIX-like operating system, including Mac OSX and Linux on Intel/ AMD x86 and ARM (e.g., Raspberry Pi) systems.


![[ka9q-radio-image.png]]

# Multicast

It's taken quite a beating in recent years, but I’m old fashioned enough to still believe in the “UNIX Philosophy”: each program should do one thing and do it well, with simple interfaces that can be used in novel ways the author may not have anticipated. The UNIX ‘pipeline’ was a seminal IPC (inter-process communication) scheme later extended to the Internet by TCP/IP.

UNIX pipes (and TCP connections) work well for point-to-point streams, and I’ve used them for signal processing. But they only have two endpoints, and you might want do drive several programs just as a 1 It's almost this simple. Some details are discussed later.hardware signal source can drive several loads through a splitter. You may want to start, stop or reconfigure one module (or move it to a different computer) without restarting everything else. In a high reliability application you might run the same program on two computers, one ready to take over if the other fails.

Sender flow control in one-to-many communication is problematic because one slow receiver might bog down others. Fortunately this isn’t necessary. A real time system processes data at a well defined rate, usually defined by an A/D converter clock. Buffering can handle momentary scheduling delays and jitter, but listeners simply must keep up on average. This makes sender flow control unnecessary.  Only listener flow control is needed, i.e., a listener must wait for a unit of data, process it, and wait for
the next unit. This simplifies things a lot.

GNU Radio already provides very flexible interconnections between signal processing modules within a single (large) program; in fact, it uses UNIX pipes internally. But I’m trying to solve a different problem: interconnecting signal processing modules that have different authors, are written in different programming languages, each with its own libraries and APIs, and run on different computers, hardware and operating systems. The Internet Engineering Task Force, the Internet protocol standards body, has (or had) a rule to standardize only “bits on wires”, i.e., actual network protocols; APIs and the like were implementation details considered out of scope. Another goal was scalability. These wise
choices -- standardizing just enough and no more -- led to the Internet's near-universal adoption by just about every computer, operating system and application.

IP is more flexible than GNU Radio’s IPC but it is also more costly. It would be wasteful to have a UDP/IP link from an oscillator to a multiplier (mixer) and another from the mixer to a detector, for example. But it can be used quite effectively at higher levels, e.g., from an SDR front end to a software tuner/demodulator, or from a tuner/demodulator to various recording and digital decoder programs. GNU Radio itself could receive, process and/or generate multicast IP streams, as could decoding programs like FLDIGI and WSJTX without relying on kludges like “virtual audio cables”. IP multicasting is especially useful for status and control messages so everyone can see what's going on.

---

# Sources

This is a list of the iq radio sources that are currently used with the ka9q-radio software.

---

# rtl-sdr

#### References [osmocom.org](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr)

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs

---

# airspy

#### References [airspy.com](https://airspy.com/)

#### Hardware description

 - Continuous 24 – 1700 MHz native RX range 
 - 3.5 dB NF between 42 and 1002 MHz
 - Maximum RF input of +10 dBm
 - Tracking RF filters
 - 35dBm IIP3 RF front end
 - 12bit ADC @ 20 MSPS (10.4 ENOB, 70dB SNR, 95dB SFDR)
 - 10MSPS IQ output
 - 80 MSPS when using custom firmware

#### Resources

#### Drivers

#### Configuration

#### Outputs

---

# airspyHF

####References [airspyHF](https://airspy.com/airspy-hf-discovery/)

#### Hardware description

 - HF coverage between 9 kHz .. 31 MHz
 - VHF coverage between 60 .. 260 MHz
 - 18 bit Embedded Digital Down Converter (DDC)
 - Up to 660 kHz alias and image free output for 768 ksps IQ

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# hackrf

#### References [hackrf](https://greatscottgadgets.com/hackrf/one/)

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# funcube_dongle

#### References [funcube_dongle](http://www.funcubedongle.com/)


#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs

---

# sockbuf

#### References

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# RX888MKII

#### References [RX888](https://groups.io/g/NextGenSDRs)

#### Hardware description

 - LTC2208 16bit ADC @ 130 MSPS
 - Dual RF input
    - HF frequency range：1kHz-64Mhz, maximum real-time bandwidth 64M. 
    - VHF frequency range：64M-1700Mhz, maximum real-time bandwidth 10M.
 - 0.5ppm VCXO
 - ATT adjustment range -32dB to 0dB
 - VGA adjustment range -11dB to +34dB
 - External 27Mhz reference clock support
 - 3.3v software switched Bias-Tee HF/VHF independent control
 - ADC PGA Rand Dither software control
 

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# SDRplay

#### References [SDRplay](https://www.sdrplay.com/)

#### Hardware description

#### Resources

 - Model Number RSP1A
 - Frequency Range 1kHz – 2GHz
 - Antenna Connector SMA
 - Antenna Impedance 50 Ohms
 - Operating Temperature Range -10˚C to +60˚C
 - Current Consumption (Typical) 185mA
 - Case Size 95mm X 80mm X 30mm
 - Case Weight 110 grams
 - USB Connector USB Type B
 - Maximum Input Power 
    - +0dBm Continuous
    - +10dBm Short Duration
 - ADC Sample Rates 2MSPS – 10.66MSPS
 - ADC Number of Bits 14bit 2MSPS – 6.048MSPS
 - 12bit 6.048MSPS – 8.064MSPS
 - 10bit 8.064MSPS – 9.216MSPS
 - 8bit >9.216MSPS
 - Bias – T 4.7V
 - 100mA Guaranteed
 - Reference 0.5ppm 24MHz TCXO. 
 - Frequency error trimmable to 0.01ppm in field.

#### Drivers

#### Configuration

#### Outputs


---

# Horus

#### References

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# Services

### Control


### Monitor


### Radio




---
# Sinks

### Opus

### Recording

---

# References

---

**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico.

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://tapr.org/37th-arrl-and-tapr-digital-communications-conference/). 40th ARRL and TAPR Digital Communications Conference, Virtual On-Line.




