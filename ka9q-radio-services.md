
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

This program allows a virtual receiver to be controlled, created or disabled under "radiod".  When in the program, press "h" to show the commands and "x" to hide them - and then "q" again to exit the program.  

- Usage:    control -s <ssrc> <multicast> <-v>

- The parameters and their defaults are:
- Where "-v" enables verbose mode.
- Where "multicast" is the name of the status (also control) stream defined in the "global" portion of the radiod .conf file:  In the example given on the rx888d page using "radiod@hf.conf" this would be "hf.local"
- Where "ssrc" (Stream Source identifier) represents the "sub channel" within the pcm multicast stream related to the status/control stream.  This could represent an existing receiver, or a previously-unused SSRC may be specified to create a new, virtual receiver - see below.
- The "ssrc" is typically defined from the frequency and "multicast" is either the numerical address - or, if you have installed Avahi, you can use the name defined in the "data" statement (e.g. "wspr-pcm.local", "wwv-pcm.local", etc.)

- Example:  control -s  5000  hf.local - This will control the 5 MHz WWV receive channel defined in "radiod@hf.conf".
- Unless explicitly defined, the ssrc will consists of the digits used to define the frequency - with the NON NUMBERS removed.
- In "radiod@hf.conf" we see that the 5 MHz WWV receiver's frequency is defined as "5000k" meaning 5000 kHz - so the SSRC is "5000"
- Alternatively, 5 MHz could have been defined as "5000000", or "5M" in which case the SSRC would have been "5000000" or "5" respectively.
- Trailing zeroes are NOT trucated.  Be careful, though:  If you defined 5 MHz as "5M000" you would have "5000" as the SSRC - the same as if you had done "5000k"!
- Similarly, a frequency defined as "12m345670" would result in an ssrc of "12345670" - again, simply by removing everything from the string but the numbers.  Specifying as "12m34567" would result in the same frequency, but the ssrc would be "1234567", instead.

- Creating a "new" virtual receiver:

- If one specifies an SSRC that does not already exist, a new receiver can be created and configured using the commands described below.  When you pick a new SSRC, be sure to use a numbering convention that is not likely to conflict with existing SSRCs defined in the .conf file.

- If you wish to disable a receiver, tune its frequency to "0" and exit.

** Using "control":

The control program can tune, create or disable a virtual receiver.  In the example above, if we had specified "1234" as the SSRC - and we knew that this wasn't defined as a receiver within our configuration file (radiod@hf.conf) we could summon a receiver into existence.

When started you will see a text-based control screen that shows the current receiver configurations and status and pressing "h" will show the list of available commands.

IMPORTANT:
- The system "locale" setting will determine how the interactive display of this program is rendered - see the file "/docs/notes.md" for more details.  For example, from the command line "export LANG=en_US.UTF-8" is known to work and is likely the default for systems set up for use in the U.S.
- Virtual receivers running under "radiod" can only be tuned within the constraints of the hardware.  For example, in the case of "radiod@hf.conf" using the RX8888 - where our sample rate is 64.8 - - MHz - we can reasonably expect to be able to tune to no more than half that frequency (e.g. it essentially covers 0-30 MHz - all of HF).  Because we are using direct sampling - that is, inhaling the entire HF spectrum, we can tune anywhere in the range that our sample rate (and any antenna path filtering) allows.
- If we used receiver hardware such as the RTL-SDR - which is a not direct-sampling receiver - our tuning range is constained.  If we have our RTL-SDR local oscillator tuned to 100 MHz and are operating with a sample rate of 2 MHz we can tune only over a 2 MHz span centered at 100 MHz - that is, 99-101 MHz.  To tune outside that range would require retuning the receiver's local oscillator - and if there are other receivers defined that require the local oscillator tuned to 100 MHz, moving it may place the received signal outside the 2 MHz range defined by our sample rate

** Overview of main display screen:

- Using the 5 MHz WWV example with "radiod@hf.conf" and the RX888 for our example, in the upper half of the screen on the left side we see:
- Carrier - This is where the receiver is tuned - 5 MHz in this case
- First LO - Using the RX-888, this shows zero since we are using direct sampling (e.g. inhaling the entire HF spectrum).  If we were using a receiver like an RTL-SDR or an SDRPlay that contains a frequency converter, we would see the center frequency of that converter here.
- IF - This is zero since our use of a direct sampling receiver doesn't need an IF.  SDRs with a frequency converter might use an IF.
- Filter low and Filter high - This defines the band-pass filter of our virtual receiver.  In our example, this is +/- 5000 Hz (5 kHz) for a total bandwidth of 5 kHz.
- Shift - If used, this will show the "shift" value defined in the .conf file for our receiver's mode as defined in "modes.conf".  As an example, the "cw" modes use this shift parameter.
- FE filter low and FE filter high - These appear to represent the usable passband of the acquisition device - More information needed
- In the left-center column we see:
- Envelope - Since we are using AM, we are using an envelope detector:  See "modes.conf" for the definition of the "AM" mode.  It is underlined, therefore enabled.
- Linear+Env - This is likely supposed to say "Linear+Envelope" (Text trucated by the display)  All modes aside from those like FM are "linear" in that they represent the amplitude of the signal in some way on the output audio stream.
- Linear - An indication that we are configured for a "linear" mode (see above)
- I/Q - Likely a status indicator showing if we are using I/Q (2-channel) rather than demodulated single channel audio - and we are using the latter.
- PLL Off - The PLL (Phase Locked Loop) can be used to recover a carrier (e.g. "Synchronous AM" - which is available in ka9q-radio) - but since we are using normal enevlope detection, it is turned off (e.g. the underline).
- PLL On - See above.  (Not enabled in our example)
- AGC Off - AGC (Automatic Gain Control) is that feature which keeps the received audio level constant despite the RF signal varying.
- AGC On - This is underlined, showing that the AGC is on.

** In the right-center column we see:
- G5RV RX888 - This shows the receiver configuration name ("G5RV" in this case) and the receiver type ("RX888") currently in use as defined in the .conf files.
- A Gain - Not known at this time.
- A/D  - This shows the total input power applied to the A/D converter relative to full-scale "dbFS".  A reading of "0" would likely indicate overload.  Under normal conditions, a reading of -10 or lower assures that brief overload events (e.g. noise peaks, lightning crashes, simultaneous signal peaks) are unlikely to happen:  Overloads can never be completely be avoided and if they are a very small proportion of the total, they are unlikely to cause degradation of received signals - particularly if their bandwidth is a tiny portion of the A/D sample rate.
- In Gain - Not known at this time
- Input - Not known at this time, but it appears to be very close to that displayed under "A/D" and is likely related to scaling/filtering/decimation?
- Baseband - After it has been filtered, this is the power within the baseband - in our case, the 10 kHz wide receiver tuned to WWV.
- NO - This is the noise density in dB/Hz in the receive path.
- S/NO - This is the ratio of the signal to noise density (I think)
- NBW - Likely related to the noise bandwidth - More details needed
- SNR - This is the instantaneous signal/noise ratio of the demodulated signal. (Need details on how this is derived)
- Gain - Not sure, but this may be the current AGC gain for the virtual receiver.
- Output - This is effectly the audio output level of the virtual receiver: It should be confortably below 0 dBFS (full-scale)
- Headroom - This is the "headroom" setting within the mode definition in "modes.conf"

** In the far-right column:
- Threshold - This is the AGC threshold, defined in modes.conf
- Recovery rate - This is the AGC recovery rate, defined in modes.conf
- Hang time - This is the AGC "hang" time, defined in modes.conf
- In the lower half of the screen under "Front end status" in the left column we see:
- Fs in - The current sample rate in samples/second
- FS out  - The output sample rate of our virtual receiver
- Block time - Raw samples from the receiver are processed in discrete, overlapping blocks.  In the example, 20 milliseconds is used meaning that new data is processed 50 times/second
- FFT in - This shows the input block size.  Since our sample rate in the example is 64800000 samples/second and we process them 50 times/second, each block is therefore 16200000 samples.
- FFT out - This is the output block size.  Since out ourput sample rate is 12000, each output block at 50 blocks/second is 300 samples.
- Overlap - This is shown in percent, so 20% of our "new" data is actually old data on our FFT blocks.
- Freq bin - This is shown in Hz, and our example shows 40 Hz FFT bin size.
- Kaiser beta - This is related to the windowing function used in the FFT.
- Drops - The number of dropped samples

** In the center column we see:
- Time and date
- Information about our server
- The current multicast control address/port
- stat pkts - The status packets from our receiver
- ctl pkts - The packets used to control our receiver
- pkts - The running total of packets from the receiver hardware
- samples - The running total of samples from the receiver hardware
- drops - The number of known-dropped packets from the receiver hardware
- dupes - The number of known duplicated packets from the receiver hardware

** In the left column we see:
- Information about our virtual receiver server/stream:  It could be a different server than that to which the receive hardware is connected
- stat pkts - Number of packets conveying virtual receiver status
- ctl pkts - Number of packets used to control our virtual receiver
** The multicast address of the pcm strem of our virtual receiver
- ssrc - The SSRC of our virtual receiver (5000 in our example, above)
- pkts - The number of packets used to convey our demodulated audio

Pressing the "h" or "?" key will toggle a screen that shows the various controls available:
- l - (lower-case "L") - Lock/unlock tuner
- Tab/PgDn - Move cursor down
- Shift+Tab/PgUp - Move cursor up
- HOME - Move cursor to top
- END - Move cursor to bottom
- Left arrow/BKSP - Move cursor left
- Right arrow - Move cursor right
- Up arrow - Increase parameter under cursor by 1
- Down arrow - Decrease parameter under cursor by 1
- CTL-L - (Control-L) - Redraw screen
- f - Set frequency:  Use value in Hz, or include modifiers as decimal place, as in:  5000000, 5m0 5000k
- g - Set gain in dB
- A - Set AGC attack rate in dB/sec
- R - Set AGC recovery rate in dB/sec
- T - Set AGC hang time in sec
- H - Set headroom in dB
- L - Set AGC threshold in dB
- P - Set PLL bandwidth in Hz
- k - Set Kaiser window beta parameter
- m - Set demod mode
- o - Set/clear option flag (What is this?)
- r - Set refresh rate in seconds
- s - Set squelch, dB
- q - quit

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




