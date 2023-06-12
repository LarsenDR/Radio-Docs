
airspyd

Usage:    airsply [-v] [-f config_file]

The parameters and their defaults are:
-v:  Verbose mode
-f:  Config file.  (Default:  "airspyd.conf")
This reads from the Airspy SDR, sending status and accepting control commands via its UDP socket - can be run as a service.  Example needed.

See "airspy.conf" for configuration example.

aprs

Usage:     aprs [-L latitude] [-M longitude] [-A altitude] [-s sourcecall] [-v] [-I mcast_address]

The parameters and their defaults are:
-v:  Verbose mode
-L:  The latitude in dd.dd format.  Use positive for NORTH latitude and negative for SOUTH latitude (e.g. 32.860400, -38.35346)
-M:  The longtidude in dd.dd format.  Use positive for EAST longitude and negative for WEST longitude (e.g. 1.5323, -117.188900)
-A:  The altitude in meters (e.g. 0.000000)
-s:  The source callsign (Default:  null)
-R:  Destination IP address (Default:  127.0.0.1:4533)
-I:  Muilticast address with in the form of:  <aaa.bbb.ccc.ddd:s> with "s" being the ssrc of the source stream.  (Default: ax25.local:5004)
Source code says "Process AX.25 frames containing APRS data, extract lat/long/apltitude, compute az/el" - see "aprs.conf" for configuration - can be run as a service.

To do:  Example showing usage

aprsfeed

Usage:     aprsfeed -u user [-p passcode] [-v] [-I mcast_address][-h host]

The parameters and their defaults are:
-v:  Verbose mode
-u:  User
-p:  Passcode
-h:  Host  (Default:  noam.aprs2.net)
-I:  Multicast address (Default:  ax25.mcast.local)
Source code says "Process AX.25 frames containing APRS data, feed to APRS2 network"" - see "aprs.conf" for configuration - can be run as a service.

To do:  Example showing usage 

control

This program allows a virtual receiver to be controlled, created or disabled under "radiod".  When in the program, press "h" to show the commands and "x" to hide them - and then "q" again to exit the program.  

Usage:    control -s <ssrc> <multicast> <-v>

The parameters and their defaults are:
Where "-v" enables verbose mode.
Where "multicast" is the name of the status (also control) stream defined in the "global" portion of the radiod .conf file:  In the example given on the rx888d page using "radiod@hf.conf" this would be "hf.local"
Where "ssrc" (Stream Source identifier) represents the "sub channel" within the pcm multicast stream related to the status/control stream.  This could represent an existing receiver, or a previously-unused SSRC may be specified to create a new, virtual receiver - see below.
The "ssrc" is typically defined from the frequency and "multicast" is either the numerical address - or, if you have installed Avahi, you can use the name defined in the "data" statement (e.g. "wspr-pcm.local", "wwv-pcm.local", etc.)

Example:  control -s  5000  hf.local - This will control the 5 MHz WWV receive channel defined in "radiod@hf.conf".
Unless explicitly defined, the ssrc will consists of the digits used to define the frequency - with the NON NUMBERS removed.
In "radiod@hf.conf" we see that the 5 MHz WWV receiver's frequency is defined as "5000k" meaning 5000 kHz - so the SSRC is "5000"
Alternatively, 5 MHz could have been defined as "5000000", or "5M" in which case the SSRC would have been "5000000" or "5" respectively.
Trailing zeroes are NOT trucated.  Be careful, though:  If you defined 5 MHz as "5M000" you would have "5000" as the SSRC - the same as if you had done "5000k"!
Similarly, a frequency defined as "12m345670" would result in an ssrc of "12345670" - again, simply by removing everything from the string but the numbers.  Specifying as "12m34567" would result in the same frequency, but the ssrc would be "1234567", instead.

Creating a "new" virtual receiver:

If one specifies an SSRC that does not already exist, a new receiver can be created and configured using the commands described below.  When you pick a new SSRC, be sure to use a numbering convention that is not likely to conflict with existing SSRCs defined in the .conf file.

If you wish to disable a receiver, tune its frequency to "0" and exit.

Using "control":

The control program can tune, create or disable a virtual receiver.  In the example above, if we had specified "1234" as the SSRC - and we knew that this wasn't defined as a receiver within our configuration file (radiod@hf.conf) we could summon a receiver into existence.

When started you will see a text-based control screen that shows the current receiver configurations and status and pressing "h" will show the list of available commands.

IMPORTANT:
The system "locale" setting will determine how the interactive display of this program is rendered - see the file "/docs/notes.md" for more details.  For example, from the command line "export LANG=en_US.UTF-8" is known to work and is likely the default for systems set up for use in the U.S.
Virtual receivers running under "radiod" can only be tuned within the constraints of the hardware.  For example, in the case of "radiod@hf.conf" using the RX8888 - where our sample rate is 64.8 MHz - we can reasonably expect to be able to tune to no more than half that frequency (e.g. it essentially covers 0-30 MHz - all of HF).  Because we are using direct sampling - that is, inhaling the entire HF spectrum, we can tune anywhere in the range that our sample rate (and any antenna path filtering) allows.
If we used receiver hardware such as the RTL-SDR - which is a not direct-sampling receiver - our tuning range is constained.  If we have our RTL-SDR local oscillator tuned to 100 MHz and are operating with a sample rate of 2 MHz we can tune only over a 2 MHz span centered at 100 MHz - that is, 99-101 MHz.  To tune outside that range would require retuning the receiver's local oscillator - and if there are other receivers defined that require the local oscillator tuned to 100 MHz, moving it may place the received signal outside the 2 MHz range defined by our sample rate

Overview of main display screen:

Using the 5 MHz WWV example with "radiod@hf.conf" and the RX888 for our example, in the upper half of the screen on the left side we see:
Carrier - This is where the receiver is tuned - 5 MHz in this case
First LO - Using the RX-888, this shows zero since we are using direct sampling (e.g. inhaling the entire HF spectrum).  If we were using a receiver like an RTL-SDR or an SDRPlay that contains a frequency converter, we would see the center frequency of that converter here.
IF - This is zero since our use of a direct sampling receiver doesn't need an IF.  SDRs with a frequency converter might use an IF.
Filter low and Filter high - This defines the band-pass filter of our virtual receiver.  In our example, this is +/- 5000 Hz (5 kHz) for a total bandwidth of 5 kHz.
Shift - If used, this will show the "shift" value defined in the .conf file for our receiver's mode as defined in "modes.conf".  As an example, the "cw" modes use this shift parameter.
FE filter low and FE filter high - These appear to represent the usable passband of the acquisition device - More information needed
In the left-center column we see:
Envelope - Since we are using AM, we are using an envelope detector:  See "modes.conf" for the definition of the "AM" mode.  It is underlined, therefore enabled.
Linear+Env - This is likely supposed to say "Linear+Envelope" (Text trucated by the display)  All modes aside from those like FM are "linear" in that they represent the amplitude of the signal in some way on the output audio stream.
Linear - An indication that we are configured for a "linear" mode (see above)
I/Q - Likely a status indicator showing if we are using I/Q (2-channel) rather than demodulated single channel audio - and we are using the latter.
PLL Off - The PLL (Phase Locked Loop) can be used to recover a carrier (e.g. "Synchronous AM" - which is available in ka9q-radio) - but since we are using normal enevlope detection, it is turned off (e.g. the underline).
PLL On - See above.  (Not enabled in our example)
AGC Off - AGC (Automatic Gain Control) is that feature which keeps the received audio level constant despite the RF signal varying.
AGC On - This is underlined, showing that the AGC is on.
In the right-center column we see:
G5RV RX888 - This shows the receiver configuration name ("G5RV" in this case) and the receiver type ("RX888") currently in use as defined in the .conf files.
A Gain - Not known at this time.
A/D  - This shows the total input power applied to the A/D converter relative to full-scale "dbFS".  A reading of "0" would likely indicate overload.  Under normal conditions, a reading of -10 or lower assures that brief overload events (e.g. noise peaks, lightning crashes, simultaneous signal peaks) are unlikely to happen:  Overloads can never be completely be avoided and if they are a very small proportion of the total, they are unlikely to cause degradation of received signals - particularly if their bandwidth is a tiny portion of the A/D sample rate.
In Gain - Not known at this time
Input - Not known at this time, but it appears to be very close to that displayed under "A/D" and is likely related to scaling/filtering/decimation?
Baseband - After it has been filtered, this is the power within the baseband - in our case, the 10 kHz wide receiver tuned to WWV.
NO - This is the noise density in dB/Hz in the receive path.
S/NO - This is the ratio of the signal to noise density (I think)
NBW - Likely related to the noise bandwidth - More details needed
SNR - This is the instantaneous signal/noise ratio of the demodulated signal. (Need details on how this is derived)
Gain - Not sure, but this may be the current AGC gain for the virtual receiver.
Output - This is effectly the audio output level of the virtual receiver: It should be confortably below 0 dBFS (full-scale)
Headroom - This is the "headroom" setting within the mode definition in "modes.conf"
In the far-right colum:
Threshold - This is the AGC threshold, defined in modes.conf
Recovery rate - This is the AGC recovery rate, defined in modes.conf
Hang time - This is the AGC "hang" time, defined in modes.conf
In the lower half of the screen under "Front end status" in the left column we see:
Fs in - The current sample rate in samples/second
FS out  - The output sample rate of our virtual receiver
Block time - Raw samples from the receiver are processed in discrete, overlapping blocks.  In the example, 20 milliseconds is used meaning that new data is processed 50 times/second
FFT in - This shows the input block size.  Since our sample rate in the example is 64800000 samples/second and we process them 50 times/second, each block is therefore 16200000 samples.
FFT out - This is the output block size.  Since out ourput sample rate is 12000, each output block at 50 blocks/second is 300 samples.
Overlap - This is shown in percent, so 20% of our "new" data is actually old data on our FFT blocks.
Freq bin - This is shown in Hz, and our example shows 40 Hz FFT bin size.
Kaiser beta - This is related to the windowing function used in the FFT.
Drops - The number of dropped samples
In the center column we see:
Time and date
Information about our server
The current multicast control address/port
stat pkts - The status packets from our receiver
ctl pkts - The packets used to control our receiver
pkts - The running total of packets from the receiver hardware
samples - The running total of samples from the receiver hardware
drops - The number of known-dropped packets from the receiver hardware
dupes - The number of known duplicated packets from the receiver hardware
In the left column we see:
Information about our virtual receiver server/stream:  It could be a different server than that to which the receive hardware is connected
stat pkts - Number of packets conveying virtual receiver status
ctl pkts - Number of packets used to control our virtual receiver
The multicast address of the pcm strem of our virtual receiver
ssrc - The SSRC of our virtual receiver (5000 in our example, above)
pkts - The number of packets used to convey our demodulated audio
Pressing the "h" or "?" key will toggle a screen that shows the various controls available:
l - (lower-case "L") - Lock/unlock tuner
Tab/PgDn - Move cursor down
Shift+Tab/PgUp - Move cursor up
HOME - Move cursor to top
END - Move cursor to bottom
Left arrow/BKSP - Move cursor left
Right arrow - Move cursor right
Up arrow - Increase parameter under cursor by 1
Down arrow - Decrease parameter under cursor by 1
CTL-L - (Control-L) - Redraw screen
f - Set frequency:  Use value in Hz, or include modifiers as decimal place, as in:  5000000, 5m0 5000k
g - Set gain in dB
A - Set AGC attack rate in dB/sec
R - Set AGC recovery rate in dB/sec
T - Set AGC hang time in sec
H - Set headroom in dB
L - Set AGC threshold in dB
P - Set PLL bandwidth in Hz
k - Set Kaiser window beta parameter
m - Set demod mode
o - Set/clear option flag (What is this?)
r - Set refresh rate in seconds
s - Set squelch, dB
q - quit


cwd

Usage:    cwd [-v] [-I fifo_name] [-s ssrc] -R mcast_group [-S speed_wpm] [-P pitch_hz] [-L level16]

The parameters and their defaults are:
-v:  Verbose mode
-I:  FIFO name (Default:  "/run/cw/input")
-s:  ssrc of on multicast address
-R:  Receive multicast address
-S:  Speed in words-per-minute (WPM) (Default:  18.0 WPM)
-P:  Pitch in Hz (Default:  500.0 Hz)
-L:  CW Level in dB (Default:  -29.0 dB)
This is a CW decoder - it may be run as a service, see "cwd.service".

To Do:  Show example of how to use CW decoder.


ft8-decoded

Usage:    ft8-decoded [-L locale] [-v] [-k] [-d recording_dir] -t [cycle_time] -c [decode_command] -l [transmission_length] PCM_multicast_address

The parameters and their defaults are:
-v:  Verbose mode
-k:  Keep .wav file after cycle (this can fill up your drive very quickly unless managed!)
-t:  Cycle time in seconds.  Examples:  15 for FT-8, 120 for WSPR
-c:  Decode command.  (Default:  "decode_ft8 %s")
-d:  Specify directory into which subdirectories of recordings will be placed.
-L:  System locale
PCM_multicast_address:  This is the multicast group address of the receiver/frequencies defined in the relevant radio.conf file.

This will decode FT8 transmissions on the defined streams - Can be run as a service.

Requires "decode_ft8" to work - possibly that from:  https://github.com/kgoba/ft8_lib - but in testing, I can't get it to execute, so I'm clearly missing something

To Do:  Determine if command-line usage is possible, configuration to allow operation as a service.

funcubed

Usage:     funcubed [-v] [-f config_file] [-N name] [-L]

The parameters and their defaults are:
-v:  Verbose mode
-f:   Config file (Default:  "funcubed.conf")
-N:  Name
This reads from the Funcube dongle SDR, accepting control commands from the UPD socket.  The default configuration is in "funcubed.conf" - Can be run as a service.  Example needed.


iqplay

Usage:    iqplay [-v] [-A iface] [-D output] [-R status] [-S ssrc] [-p TOS] [-T ttl] [-b blocksize] [-f frequency]

The parameters and their defaults are:
-v:  Verbose mode
-A:  Default interface (e.g. Eth0)
-D:  Output multicast address
-R:  Status  (Default:  Uses time for "gps_time_sec()")
-S:  ssrc of produced multicast stream
-p:  IP Type of Service
-T:  Multicast Time-To-Live
-b:  Block size
-f:  Frequency, in Hz - used to indicate the frequency when "STDIN" is used, or if there's no tag on the file

Source code says:  "Read from IQ recording, multicast in (hopefully) real time"

To Do:  Figure out how to use it and for what it might be useful.


iqrecord

Source code says "Reads and records complex I/Q stream or PCM baseband audio" - "NOT CURRENTLY USABLE - needs to read status stream to get sample rate, etc."

Usage:    iqrecord multicast_addr -S status_addr [-r samp_rate] [-d duration_sec] [-D dir_name] [-l locale] [-d duration] [-q] [-v]

The parameters and their defaults are:
-v:  Verbose mode
-q:  Quiet mode (Suppress display)  (Default off - Do not suppress display)
-r:  Sample Rate
-d:  Duration, in seconds
-D:  Directory name
-l:  Locale
-S:  Status address


metadump

This may be used to display the current status of the receiver, using the control/status multicast stream.

Usage:    metadump [-v] multicast address

The parameters and their defaults are:
-v:  Verbose mode
multicast address:  Multicast address of the status/control stream, not the data stream.

Example:

    metadump rx888-status.local - This shows the current status of the RX-888 receiver, sending it to STDOUT.


modulate

Source code says: "Simple I/Q AM modulator - will eventuall support other modes".

Usage:    modulate [-v] [-r sample_rate] [-f frequency] [-a amplitude] [-s sweep_rate] [-m modulation_type] [-W wisdom_file]

The parameters and their defaults are:
-v:  Verbose mode
-r:  Sample rate in Hz  (Default:  192kHz)   The input and output rates are always the same.
-f:  Frequency in Hz of modulated carrier  (Default:  48 kHz)
-a:  Amplitude in dB relative to full scale (Default:  -20 dBFS)
-s:  Sweep rate, in Hz/second  (Default:  0 - disabled)
-m:  Modulation type  (Default:  "am" - This may be the only mode currently supported, but it looks as though the other modes are really there in the source code.)
Available modulation types:
"am" - full-carrier, +/- 5000 Hz BW
"usb" - suppressed-carrier, 0-3000 Hz BW
"lsb" - suppressed-carrier, -3000-0 Hz BW
"ame" - Upper-sideband AM, full carrier, 0-3000 Hz BW
"dsb" - Double-sideband, suppressed carrier, -5000-5000 Hz BW
-W:  Specify FFT "Wisdom" file other than default in /var/lib/kq9q-radio/wisdom - There may be a problem with this - see below.
This appears to use STDIO (pipe) for both input (modulation source) and output, both of which must be operating at the same sample rate.


Example:

sox test.wav -r 48000 -c 2 -b 16 -t wav - | modulate -v -r 48000 -a 0 -f 10000 -m usb | aplay -r 48000 -c 2 -f s16_le

In the above example:
We have a file called "test.wav" that is being played at a sample rate of 48 kHz ("-r 48000") and we use the "sox" utility to play it back using 16 bits per sample to STDOUT.  Two-channel mode ("-c 2") is used as it seems that that's what modulate wants.
The playback audio is piped to the modulate utility where we specify the same rate as the file ("-r 48000") with an amplitude of 0 dBFS ("-a 0") and specify upper-sideband ("-m usb") and a "carrier" frequency of 10 kHz ("-f 10000").
The output of this is then piped - also via STDOUT - to "aplay" where we again specify the sample rate (48 kHz), single-channel, and 16 bit data - specifically "s16_le" which is 16 bit "little-endian" - and output it to the default sound card.
Again we need to use 2-channel mode, but this is because the output is complex (I/Q).

The result of all of this is that at the sound card output we will get the contents of our .wav file modulated as USB at 10 kHz with a complex output.  This output could be directly applied to a pair of mixers driven by a local oscillator in quadrature to directly generate modulated RF.

Bugs/quirks as of this writing:
Versions prior to 9 June, 2023 had a problem in which the carrier in the AM modes ("am" and "ame" as of this writing) wasn't being generated properly.
As of the time of writing:  This program uses FFT functions and it currently does not re-use existing FFT "wisdom" data.  Because of this there will be a delay of between 5 and 60 seconds (depending the CPU capacility of your computer - typically around 8-16 seconds for a modern PC with an i5 or i7) between invocation of modulate and when it actually starts producing output.

nwisdom

The "radiod" utility uses the "FFTW3" library and this is, by far, the heaviest user of CPU in ka9q-radio.  Because it is so processor-intensive, optimizations are possible to improve its performance and these are based specifically on the CPU environment in which ka9q-radio is used.  While it isn't strictly necessary to make these optimizations, they are particularly helpful on systems that need all of the help that they can get (e.g. older computers, Raspberry Pi).

The file "docs/FFTW3.md" is recommended reading and it includes the use of nwisdom, but the most important portion is repeated here:

FFTW3 provides a 'wisdom' feature to find and remember the most efficient way to perform specific transform types and sizes.  While radiod will probably run in real time without it on faster x86 systems it is especially recommended on the Raspberry Pi 4. This requires that you run FFTW3's fftwf-wisdom utility with the actual transform sizes needed by the parameters you use with the radiod program. This can take hours but is worth the improvement in performance.

Until I can figure out how to do all this easily and automatically, here's a suggested run:

$ time fftwf-wisdom -v -T 1 -o nwisdom rof500000 cof36480 cob1920 cob1200 cob960 cob800 cob600 cob480 cob320 cob300 cob200 cob160

This finds the best way to do the following transforms:

rof500000: Airspy R2, 20 Ms/s real, 20 ms (50 Hz) block time, overlap 5 (20%).

cof36480: Airspy HF+, 912 ks/s complex, 20 ms (50 Hz), overlap 2 (50%).

cob1920, etc: inverse FFTs for 20 ms (50 Hz) block times, overlap factors of 2 and 5, and various common sample rates supported by the Opus codec (8/12/16/24/48 kHz).

NB!! fftwf-wisdom isn't careful about permissions checking. Nor does it do any checkpointing. I've had hour-long runs ruined because it couldn't write its output file. Write into a temporary file (e.g., nwisdom) and then carefully move that into /etc/fftw/wisdomf after backing up previous versions.

Note also that wisdom files computed for the multithreading option (which I use) are NOT compatible with wisdom files computed without multithreading. That's true even for -T 1 (multithreading with just one thread). Do NOT omit the -T 1 option, or you may destroy all your previous computation work!

FFTW3 provides a way to automatically create and cache wisdom at runtime, but with a very long startup delay. I need to move wisdom generation to a separate process independent of radiod. It would be nice if FFTW3 automatically learned as it went, getting faster on user data without performing the exhaustive search up front.




opusd

Usage:      opusd [-l|-V] [-x] [-v] [-f] [-p tos] [-o bitrate] [-B blocktime] [-N name] [-T ttl] [-A iface] [-I input_mcast_address | -S input_status_address] -R output_mcast_address

The parameters and their defaults are:
-l  OR  -V:  -l = low delay mode optimization;  -V = VOIP mode optimization.  (Default with neither option is for general audio applications)
-x:  Discontinuous (e.g. "DTX" - reduced bit rate when audio is silent - Default = 0, off)
-v:  Verbose
-f:  Enable FEC (Default = 0, off)
-p:  IP Type-of-service (Default:  48, or AF12<<2)
-o: Opus bit rate (Default:  32 kbits/sec)
-B:  Blocktime in msec  (Default:  20 msec)
-N:  Name of stream  (Default:  null)
-T: Time To Live  (Default:  1)
-A:  Interface (e.g. "Eth0")
-I:  Input multicast address (Default:  null)
-S:  Input status address (Default:  null)
-R: Output Multicast Address (Default:  null)

Opus is an open-source codec package and this, along with "opussend" (below) can be used to convey one or more audio streams across the Internet - or across LAN/WAN segments that are not "friendly" to Multicast data.

This may be run as a service:  See the related "opusd.conf" files.

More information about the Opus Codec may be found here:  https://www.opus-codec.org/

To Do:  Example of use

opussend

Usage:    opussend [-L] [-x] [-v] [-o bitrate] [-B blocktime] [-I input muticast address] [-R output multicast address] [-T multicast TTL] [-p Type of Service] [-f FEC]

The parameters and their defaults are:
-o: Opus bit rate (Default:  32 kbits/sec)
-B:  Blocktime in msec  (Default:  20 msec.)
There may be some advantage of having this blocktime the same as that defined in the [global] section of radio.conf.
-T: Time To Live  (Default:  10)
-R: Output Multicast Address (Default:  null)
-p:  IP Type-of-service (Default:  48, or AF12<<2)
-v:  Verbose
-x:  Discontinuous (e.g. "DTX" - reduced bit rate when audio is silent - Default is off)
-I:  Input multicast address (Default:  null)
-L:  List audio devices
-f:  Forward Error Correction (Default:  0 - disabled)
Specifying nothing for the "-R" parameter will cause opssend to attempt to use a (default?) sound card as the output.

This utility may be used to send audio from multicast data produced by KA9Q-Radio using the Opus codec.  In this way audio can be coded and transported using TCP/IP and conveyed across the Internet - or LAN/WAN segments that are not "friendly" to Multicast data.

See "opusd" (above) for more information about the Opus codec.

To do:  Example of use


packetd

Usage:    packetd [--verbose|-v] [--ttl|-T mcast_ttl] [--pcm-in|-I input_mcast_address [--pcm-in|-I address2]] [--ax25-out|-R output_mcast_address] [input_address ...]

The parameters and their defaults are:
--verbose OR -v:  Verbose mode
--ttl OR -T:  multicast time-to-live
--pcm-in OR -I:  input multicast address
--pcm-in OR -I:  Second input multicast address  (up to 10, total)
--ax25-out OR -R:  Output multicast address
-N:  Name
-A:  Multicast interface
-S:  Status address
-p:  IP Type-of-service (Default:  0)

Source code says "Reads RTP PCM audio stream, emits decoded frames in multicast RTP"

Available as a service - uses "packetd.conf"

To do:  Example of use

pcmcat 

pcamcat send the contents of the specified stream to STDOUT.  If you need to pick off a single receiver and send it to an audio device, a file or another stream, this is one way to do it - but note that there may be other ways that might better suit your specific need.

Usage:    pcmcat <multicast addr> -s <ssrc>

The parameters and their defaults are:
-s <ssrc> - Specify ssrc, typically the text of the frequency in the .conf file with the non-numerical characters remove
-2 - Force 2-channel mode - typically used for I/Q data or FM broadcast stereo audio  (Default is 1 channel)
-v - Verbose mode
-q - "Quiet" mode (no console output
While it is possible to use other tools to extract a audio from a multicast stream (To Do:  Discuss other methods in this or another document) the "pcmcat" tool allows you to do so and send it to STDOUT.  Taking the example of the WWV receivers again, consider the following line:

./pcmcat 239.188.31.156 -s 10000 | aplay -r 12000 -c 1 -f s16_le

If you have installed "Avahi", you could alternatively used the line:  ./pcmcat wwv-pcm.local -s 10000 | aplay -r 12000 -c 1 -f s16_le

If all goes well (e.g. rx888d and radiod are running) you will hear the 10 MHz WWV audio.

In the above we see the multicast IP address for the WWV receivers - but following the "-s" parameter we see the "ssrc" - in this case "10000" representing 10 MHz (or another number if we had used the "ssrc" parameter when we defined the receiver).  Following this we see that we have piped the audio to "aplay", specifying a sample rate of 12 kHz (-r 12000), a monaural source (-c 1) and the format of our audio (-f s16_le) which is 16-bit signed, little-endian.

If desired, you could pipe the raw audio somewhere else - perhaps to a file - or use SOX to write it to a .wav file, instead as this example:

 ./pcmcat wwv-pcm.local -s 10000 | sox -t raw -r 12000 -b 16 -c 1 -L -e signed-integer - out.wav

This will record the 10 MHz WWV receiver, via "STDOUT" from pcmcat to the file "out.wav".

To make it record for 2 minutes, the following will work:

timeout 120  ./pcmcat wwv-pcm.local -s 10000 | sox -t raw -r 12000 -b 16 -c 1 -L -e signed-integer - out.wav


pcmsend

This program allows the "sending" of PCM data (audio device?) via multicast.

Usage:  pcmsend -I <audio device> -R <multicast address>

The parameters and their defaults are:
-L:  List audio devices
-p:  IP Type-of-service (Default:  48 = AF12<<2)
-T:  Multicast TTL (Default:  1)
-v:  Verbose mode
-I:  Audio device  (Default:  null)
-R:  Multicast output address  (Default:  null)
To do:  Determine proper formatting of <audio device> and give example of use


pcmrecord

This records elements of the multicast stream to individual .wav files:  If there are multiple audio channels defined in the relevant section of the "radiod.conf" file, a separate .wav file is produces for each radio channel/frequency.

Usage:  pcmrecord -s -v -d -t <timeout> -m <sec> -l <locale>   <multicast addr>

The parameters and their defaults are:
-v - Verbose mode
-l - Desired locale (default will be that to which your system is set - can affect numerical/date format)
-s - Create subdirectories for the .wave files using the ssrc in the multicast as the name (files will NOT be in subdirectories if this is not included)
-d <name> - Specify directory name.  If "-s" is used, those subdirectories are placed in the directory specified here.
-t <sec> - Timeout (in case data stops)
-m <sec> - Set minimum time for "substantial" file.  If the file created is less than the duration specified in seconds, it is automatically deleted when the stream ends:  This prevents the creation of lots small files shorter than the specified duration.
.wav files are created at the bit rate specified in the stream definition (12 kHz for the "wwv" example, above) and are named:

<frequency><YYYY-MM-DD>T<HH:MM:SS.S>Z.wav

Where "frequency" is that named in the .config file that defines the stream.  (e.g. "5000k" for the 5 MHz WWV example)

Usage examples:

    pcmrecord -s -d recordings wwv-pcm.local -v    This will create a .wav file for each element of the stream (e.g. a separate .wav file each for 2.5, 5, 10, 15, 20 and 25 MHz) and place it in the "recordings" subdirectory.

To create a timed recording (e.g. stops after a certain amount of time) precede this with "timeout <x>" where "x" is the number of seconds, as in:
   
    timeout 60 pcmrecord -s -d recordings wwv-pcm.local -v  to produce a .wav file of 60 seconds duration


pcmspawn


Source code says "Receive and demux RTP PCM streams into a command pipeline" - program has no usage/help information and I'm not sure what it does, exactly - need to find example of use-case.

Usage:  pcmspawn [-v] [-S status multicast] [-N command] [-A default interface] [-I Input multicast address]

The parameters and their defaults are:
-v:  Verbose mode
-S:  Status stream multicast
-A:  Default interface (e.g. "Eth0")
-I:  Input multicast address
-N:  Command
To do:  Give example of use - determine precise format used in "-N command".


pl

This is a PL (subaudible tone) decoder - used primarily for VHF/UHF FM reception.

Usage:  pl -A <iface> -I <multicast address> -T <ttl>  OR  pl -A <iface> -T <ttl> <multicaqst address>

The parameters and their defaults are:
-v:  Verbose mode
-A:  Interface (e.g. "Eth0")
-T:  Time to live (Default:  10)
-I:  (The "-I" parameter is optional)  Multicast address:  Several multicast addresses may be specified (Up to 20 as of this writing)

Minimal example:  pl <mcast address>

When any of the tones defined in "pl" are detected - which includes the standard CTCSS (subaudible)tones - the ssrc on which the tone was detected, the tone frequency and its level will be sent to STDOUT.  Note that if multiple receivers/frequencies are specified in the "radiod.conf" file, each audio stream is monitored for content.

It appears that the "-I" parameter is optional.  More than one multicast address may be specified.

This is useful if one has, for example, multiple 2 meter and 70cm receivers tuned to local repeaters:  With this one command one can determine when a audbaudible tones are properly decoded which, in turn, could trigger recording/logging, etc.

Example output text:  ssrc 146520: tone 100.0 Hz -5.0 dB  - This might be the output of a (hypothetical) receiver on 146.520 MHz where a 100.0 Hz tone was present.

powers

Usage:    powers [-v] [-b bins] [-c count] [-f] frequency] [-i interval] [-s ssrc] [-t tc] [-T timout] [-w Binwidth] [input multicast address]

The parameters and their defaults are:
-b:  Number of FFT bins (Default:  0)
-c:  Count of number of updates to do.  Will repeat infinitely if set to -1.  (Default:  1)
-f:  Frequency  (Default:  -1)
-i:  Interval between updates, in seconds  (Default:  5)
-s:  SSRC of input multicast stream
-t:  tc (Default:  0) - Need to determine what this is.
-T:  Retransmission timeout (Default:  No timeout)
-v:  Verbose mode
-w:  Bin bandwidth (Default:  0)

Source code says "Read FFT bin energies from spectrum pseudo-demod and format similar to rtl_power"

The command line:  powers -v -s 5000 wwv-pcm.local -f 5M seems OK in that it resolves the multicast address of the WWV receiver, but nothing appears on the screen - even with varying parameters entered in the -b (bins), -w (width), -f (frequency), -i (interval) parameters.  I'm probably doing something wrong here - likely because I have been trying it from the CLI.

radiod

This is the main processing program that takes raw samples from the acquisition device and produces the individual pcm data streams.

See the examples on the page http://www.sdrutah.org/info/using_ka9q_radio_with_the_rx888.html for an example of its use with the RX-888.


rdsd

Source code says "FM RDS demodulator/decoder".  Useful only for WFM FM broadcast signals:  RDS is a data channel found at 57 kHz on many FM broadcast signals that conveys song/title/artist information, station identification, and can also can contain other information such as traffic and weather data.

Usage:    rdsd [-v] [-T mcast_ttl] -I input_mcast_address -R output_mcast_address

The parameters and their defaults are:
-v:  Verbose mode
-T:  Multicast TTL (time to live)  (Default:  10)
-N:  Name (Default:  "rds")
-S:  Status 
-p:  IP TOS (Type of service)  (Default:  48  AF12<<2)
-I:  Input multicast address
See "rdsd.conf" for additional configuration.  This may be run as a service.
rtlsdrd

Usage:    rtlsdrd [-A mcast_ifce] [-D mcast_dest_addr] [-I rtlsdr_dev_index] [-L] [-R metadata_dest] [-S rtp_ssrc] [-T rtp_ttl] [-a] [-b] [-c] [-f freq_hz] [-h] [-p IP_tos] [-r sample_rate] [-t mcast_stat_ttl] [-v]


The parameters and their defaults are:
-v:  Verbose mode
-A:  Multicast interface (e.g. eth0)
-D:  Multicast destination address
-I:  Input device (Default is device "0")  This may allow the use of the serial number defined by the user using the "rtl_eeprom" utility.
-L:  Set "linearity" using gain tables - default without this parameter is "sensitivity"
-R:  Metadata destination address
-S:  ssrc for multicast output data
-T:  TTL (time to live) for RTP data
-a:  Enable AGC
-b:  Enable bias voltage on antenna connector
-c:  Frequency calibration
-f:  Initial center frequency in Hz
-p:  IP TOS (Type of service)  (Default:  48  AF12<<2)
-r:  Sample rate .  RTL-SDR dongles typically support sample rates of  225-300 kHz and 900-2048 kHz - the upper limit usually being limited by the USB hardware.  Sample rates of higher than 2048 may work in some cases (particularly on USB 3.0 hardware) but this would need to be tested on a per-device basis to determine if samples are dropped.
-t:  Multicast status stream TTL (time to live)
Note:  Clarification of "Linearity" and "Sensitivity" modes controlled by the "-L" parameter and how gain might be manually set is needed.  It is unknown how this module handles the use of "Converter" mode - via the R820T converter - and "Direct" mode as used <30 MHz.

This reads from an RTL-SDR dongle SDR,sending status and accepting control commands via its UDP socket.  Example needed.


rx888d

Usage:    rx888d [-v] [-f config_file]

The parameters and their defaults are:
-v:  Verbose mode
-f:  Configuration file (Default:  "rx888d.conf")

This reads from an RX-888 SDR, sending status and accepting control commands via its UDP socket - see the page http://www.sdrutah.org/info/using_ka9q_radio_with_the_rx888.html for an example of its use.

The default configuration for the rx888 may be found in the file rx888d.conf  - but note that the working version of this file will be found in /etc/radio  - so if you make any changes, you'll need to do so there, likely requiring sudo to edit it.

sdrplayd

Usage:    sdrplayd [-v] [-f config_file]

The parameters and their defaults are:
-v:  Verbose mode
-f:  Configuration file (Default:  "sdrplayd.conf")
This reads from an SDRPlay SDR, sending status and accepting control commands via its UDP socket - Can be run as a service.

NOTE:  You MUST have the SDRPlay API installed to compile!



setfilt

Usage:    setfilt [-v] [-l locale] [-r Radio]

The parameters and their defaults are:
-v:  Verbose mode
-l:  Locale (Default:  "en_us.UTF-8")
-r:  Radio (Default:  null)  I'm not sure if this is the data multicast or the status multicast.
Source code says "Interactive program to set predetection filters".  

This apparently allows the adjustment of the "low" and "high" band-pass limits of receivers, but I've not yet figured out exactly what this does or how to use it.



show_pkt

Usage:    show_pkt [-v] Multicast_addr

The parameters and their defaults are:
-v:  Verbose mode
Multicast_addr:  The multicast address of receiver (I think)
Source code says:  "Display RTP statistics"

show_sig

Usage:    show_sig [-v] Multicast_addr

The parameters and their defaults are:
-v:  Verbose mode
Multicast_addr:  The multicast address of receiver (I think)

Source code says "Display signal levels".

Program crashes with "buffer overflow detected" when invoked with either a PCM or metadata stream - I'm probably doing something wrong.


steredod

Usage:      stereod [-v] [-T mcast_ttl] [-S status_address | -I input_mcast_address]

The parameters and their defaults are:
-v:  Verbose mode
-A:  Interface (e.g. "Eth0")
-I:  Input multicast address
-N:  Name (Default:  "stereo")
-R:  Output (Multicast "send" socket)
-S:  Status
-p:  IP TOS (Type of service)  (Default:  48  AF12<<2)
-t:  Multicast status stream TTL (time to live)  (Default:  10)
--pcm-out:  Sound card/PCM device (used instead of "-R")
Stereo decoder for WFM demodulation stream:  It will send to a PCM device (or output stream?) - Can be run as a service.  See "stereod.conf".


tune

Usage:    tune [-h] [-l LOCALE] -r/--radio RADIO -s/--ssrc SSRC [-v] FREQUENCY

The parameters and their defaults are:
-v:  Verbose mode
-s or --ssrc:  ssrc of audio stream
-l:  Locale  (Default:  "en_us.UTF-8")
-r or --radio:  Radio multicast address
Source code says "Interactive program to tune radio"

A minimal example is:

    tune -r <tuner multicast> -s <ssrc> 

Note:  This may not be appropriate for a direct-sampling receiver like the RX-888 where the sample rate is fixed.  It appears to "work" - but it doesn't do anything - when "hf.local" (the "status" value in radiod@hfconf.conf) is used for the "-r" parameter.

This and/or "control" may be useful for controlling the "sub" receivers, but I haven't been able to do this so I suspect that I'm doing something wrong.


wspr-decoded

Usage:    wspr-decoded [-l locale] [-v] [-k] [-d recdir] PCM_multicast_address

The parameters and their defaults are:
-v:  Verbose mode
-l:  Locale
-d:  Record directory under which the individual receivers' files (wspr decoded, audio) go. 
-k:  Keep audio recordings after decode cycle.  (Warning:  This can quickly fill a drive!)
PCM_multicast_address:  This is the multicast address of the receiver (audio) data.
This may be used to monitor multiple outputs (frequencies) from radiod and perform WSPR decodes upon those - wsjtx is required for this to work.  This is available as a service.

Example (as a program):

    ./wspr-decoded 239.72.24.12 &      or   ./wspr-decoded wspr-pcm.local &

Note:  The "&" at the end causes it to run as a background process - use "killall wspr-decoded" to stop, but wspr-decoded is also available as a service.  The file "wspr-decoded.conf" is associated.

This will cause all defined wspr frequencies to be decoded on a 2-minute interval.  Note that this simply decodes the received signals and places the report in "ALL_WSPR.TXT".  This does NOT upload WSPR spots to wsprnet.org.

The files produced by wspr-decoded are placed in individual directories with the names bearing the receive frequencies, as in "14956000" for 20 meters and "3568600" for 80 meters.  If the "-d" parameter is specified, this will define the name of the root directory containing these sub-directories.

Within each frequency's directory are the following files:
A .wav file named:  <YYMMDD>_<HHMM>.wav  These files are approximately 1:54 in length.  (These files will pile up if the "-k" option is specified!)
ALL_WSPR.TXT - This contains the decoded spots
hashtable.txt - Used primarily for "type 2" WSPR spots where information is sent in more than one transmission, but a "hash" is used in lieu of a callsign to alow the "other" packet (without the callsign) to be associated with the one with the callsign.  Type 2 spots most commonly used to convey the 6-character grid square or additional information such as telemetry, etc. used for baloons and other special purposes.
wspr_timer.out -Contains statistics related to decoding
wspr_wisdom.dat - This is used for optimizing the FFT used for decoding.

NOTE:  The "wsprd" binary (part of wsjt-x) used to decode the packets may consume all available processing time.  What this means is that even on a fairly fast computer, audio drop-outs and lost receive data samples may be occur starting at the beginning of every even minute as WSPR spot processing is done.  It may be necessary to change the priority (e.g. set a high "nice" number) of the WSPR decoding process to prevent this from happening along with increasing the priority of other critical processes like radiod.
