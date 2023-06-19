
# Documentation of ka9q-radio

edited by Dave Larsen, KV0S




# Services

This is a list of the iq radio services that are currently used with the ka9q-radio software.



### aprs

A program to decode APRS packets.

- Usage:     aprs [-L latitude] [-M longitude] [-A altitude] [-s sourcecall] [-v] [-I mcast_address]

- The parameters and their defaults are:
- -v:  Verbose mode
- -L:  The latitude in dd.dd format.  Use positive for NORTH latitude and negative for SOUTH latitude (e.g. 32.860400, -38.35346)
- -M:  The longtidude in dd.dd format.  Use positive for EAST longitude and negative for WEST longitude (e.g. 1.5323, -117.188900)
- -A:  The altitude in meters (e.g. 0.000000)
- -s:  The source callsign (Default:  null)
- -R:  Destination IP address (Default:  127.0.0.1:4533)
- -I:  Muilticast address with in the form of:  <aaa.bbb.ccc.ddd:s> with "s" being the ssrc of the source stream.  (Default: ax25.local:5004)
- Source code says "Process AX.25 frames containing APRS data, extract lat/long/apltitude, compute az/el" - see "aprs.conf" for configuration - can be run as a service.


### aprsfeed

A program to upload APRS recived packets to a Tier2 server.

- Usage:     aprsfeed -u user [-p passcode] [-v] [-I mcast_address][-h host]

- The parameters and their defaults are:
- -v:  Verbose mode
- -u:  User
- -p:  Passcode
- -h:  Host  (Default:  noam.aprs2.net)
- -I:  Multicast address (Default:  ax25.mcast.local)
- Source code says "Process AX.25 frames containing APRS data, feed to APRS2 network"" - see "aprs.conf" for configuration - can be run as a service.


### control

A program that controls both the reciever specific program and the radiod program.

### cwd 

A program to decode CW signals in a radio stream.

- Usage:    cwd [-v] [-I fifo_name] [-s ssrc] -R mcast_group [-S speed_wpm] [-P pitch_hz] [-L level16]

The parameters and their defaults are:
- -v:  Verbose mode
- -I:  FIFO name (Default:  "/run/cw/input")
- -s:  ssrc of on multicast address
- -R:  Receive multicast address
- -S:  Speed in words-per-minute (WPM) (Default:  18.0 WPM)
- -P:  Pitch in Hz (Default:  500.0 Hz)
- -L:  CW Level in dB (Default:  -29.0 dB)
- This is a CW decoder - it may be run as a service, see "cwd.service".


### ft8-decoded

A program to decode ft8 signals it is forked from the wspr-decoded code.

### iqplay

A program to play IQ data stream

### iqrecord

A program to record rare IQ streams to memory for later post processing.

### metadump

### modulate

### monitor

A program that allows monitoring the signals as per the configuration file. 

### opussend

### packetd

### pcmcat

### pcmsend

### pcmrecord

### pcmspawn

### pl

### powers

### radiod

radiod receives signals from the hardware specific recievers and creates the connections to the various post processing uses of the radio signals.

### rdsd

### setfilt

### show_pkt

### show_sig

### stereod

A program used with FM stereo signals

### tune

### wspr-decoded
A program to decode wspr packets. 


# References



**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico.

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://tapr.org/37th-arrl-and-tapr-digital-communications-conference/). 40th ARRL and TAPR Digital Communications Conference, Virtual On-Line.




