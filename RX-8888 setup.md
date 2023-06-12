# Example of a radio hardware .conf file  
  
Note:  The working versions of these files are located in the /etc/radio directory.  They will exist in the "ka9q-radio" directory as well, but if you make changes to those files, nothing will happen - ONLY the changes that you make to the files in /etc/radio will have an effect!  
  
To check:  It appears that the files in /etc/radio may get overwritten when updated from github and/or when one invokes the Makefile.  
  
Let us consider the file "rx888d.conf":  

```
[g5rv]  
# ka9q customized  
description = "G5RV RX888"  
firmware = SDDC_FX3.img  
samprate = 64800000    ;  2^8 * 3^4 * 5^5  
# needs fftw3 wisdom and/or fft-threads >= 4 and some buffer tuning  
# seems to lose data in the network path  
# forward FFT for 129,600,000 Hz, 20ms and overlap = 5 is 3240000  
#samprate = 129600000    ;  2^9 * 3^4 * 5^5  
iface = eth0               ; force primary interface, avoid wifi  
status = rx888-status.local  
data = rx888-pcm.local  
ssrc = 10  
;gain = 1.5 ; dB  
gain = 10 ;dB - near floor of NF curve, still not too high for my G5RV  
gainmode = high ; higher gain range  

```


Note:  For the radio hardware configuration file, a number of the parameters are specific to the receive hardware being used and their availability and usefulness will vary depending on that hardware.  For example, the rx888d does not have a tuned frequency like other hardware (e.g. airsply, RTL-SDR dongle, SDRPlay) as it - when used on HF - can simply "inhale" the entire spectrum, making "tuning" irrelevant.  
  
In the above we see the heading of the section [g5rv]:  This name is arbitrary and in this case, it refers to a hypothetical antenna and the configuration one might use with that antenna.  If you want to experiment with different hardware configurations, you can add additional sections below this.  
  
In this case, this configuration file is invoked as:  ./rx888d g5rv  
  
or, as a services, as:  
  
systemctl start rx888d@g5rv.service  
  
About the fields:  

- description - It's recommended that you include a description of the purpose of this definition.  Question:  Is this field used anywhere else?
- firmware - This is specific to the "rx888" receiver and it tells the driver which version of firmware to load - other receive hardware may not need this.
- samprate - This is the acquisition sample rate of the receiver hardware - and this is likely going to be many time greater than the sample rate of any single virtual receiver instance.  In the case of the RX888 - which can receiver the entire HF (+6 meter) spectrum all at once, we have a rather high sample rate - in this case, 64.8 Msps 

- Note:  There may be certain mathematical/algorithmic restrictions on the precise sample rates with respect to the desired output sample rates, overlap size and block time (used in the "radiod" configuration) - I need to look into those to see what - if any - these might be to determine possible impact on performance.

- For other receiver hardware (e.g. AirSpy, RTL-SDR, SDRPlay, HackRF) this sample rate is typically in the megasamples-per-second region to allow an entire band to be "inhaled" and tuned by radiod.

- In some cases one must select a "bandwidth" setting for the hardware that is appropriate to the sample rate chosen.

- In the case of the SDRPLay receivers, one may select 200, 300, 600, 1536, 5000, 6000, 7000 and 8000 kHz filters and one would typically choose a sample rate a bit higher than one of these bandwidths.

- If, while using the SDRPlay, you chose a sample rate of, say, 3200 kHz, you would not want to use the 1536 kHz filter as that would filter out about half the signals, effectively wasting processor power.  If you chose a 5000 kHz filter, instead - which is wider than 3200 kHz - you would experience "aliasing" (false signals) from beyond the edges of the passband.  If you needed to cover 3200 kHz of passband, you would be better off selecting the 5000 kHz filter and a suitable sample rate (perhaps 5300 kHz) and accept the higher sample rate and CPU utilization.

- The R820T "tuner" portion of the RTL-SDR has built-in filtering and thus bandwidth selections that should be chosen appropriately with respect to sample rate.

- When in "direct" mode the R820T tuner is not used and there are no options for bandwidth selection.  
    

- iface - As noted in the documentation, ka9q-radio communicates between the various pieces (e.g. radio hardware to radiod demodulators, the outputs of the demodulators, etc.) using multicast, and this specifies the interface for that data.

- Note that in many cases that a local loopback is used rather than an actual Ethernet interface:  In the case here - with a sample rate of 64.8 Msps - this is too fast even for a Gig-E port, so we aren't going to traverse our LAN with such data rates.
- In the case of another types of radio hardware - such as an AirSpy, RTL-SDR or SDRPlay, our rate will be much lower and we could convey our raw data across a LAN to one or more receivers runing "radiod" if we wished to do so.

- status - This defines the multicast host name of the status and control stream for this receiver.
- data - This defines the multicast host name of the raw receive data for this stream.
- ssrc - As noted above, ka9q-radio uses Multicast to convey data - and in a given multicast stream one can have several different streams.  The "ssrc" - (variously called "Synchronization Source" or "Stream Source Identifier") is used to identify a "sub" stream within a multicast stream.  In this case we are tagging our raw receive data with an ssrc of "10", but it could be any 32 bit integer.  We'll see many more of these SSRCs later!
- gain - This is a hardware-specific parameter for the receiver we are using.  In this case - for the RX888 - we are specifying a gain of 10 dB for the receiver.

- When setting the gain, there is a delicate balance between having too little signal (weak signals being lost in the receiver noise) and having too much signal (overload!).  While setting this parameter properly depends greatly on your receiver antenna system, ka9q-radio contains tools (e.g. the "control" program) that may be used to observe levels and make corrections as needed.

- gainmode - This is a hardware-specific parameter for the receiver that we are using.  In the case of the RX888 not only can one adjust the gain (attenuator, actually) with the "gain" parameter, above, but one can specify if the gain mode is "high" or "low" - like a preamplifier.  To a degree, "gain" and "gainmode" are a bit redundant here, but since the hardware supports both, so does the driver.

- For the RX888 Mk2 an additional gain control (attenuator, actually) was added - but the "high/low" gain setting of the original RX888 was retained, hence both parameters.  
    

More than one configuration  
  
It is possible to have more than one configuration within this .conf file.  For example, if you were experimenting with different gain values, antennas, or sample rates, you could define such.  For example, if you wanted to have a section called "[dipole]", you would simply repeat the configuration under the [g5rv] section and make the desired changes, and when you invoked it you would simply substitute "g5rv" for "dipole" as follows:  
  
./rx888d dipole  
  
or, as a services, as:  
  
systemctl start rx888d@dipole.service  
  

---

  
Example of a "radiod" configuration file  
  
While the "hardware" configuration file is specific to the SDR hardware in question, the "radiod" configuration is not.  This provides a layer of abstraction in that as long as your raw data from the receiver contains the frequency-related signal data that we want, the configuration and operation of "radiod" is more or less agnostic to your receiver hardware.  In other words, if you were to have configured a set of receiver configurations with an Airspy, RTL-SDR or SDRPlay receiver, you can likely replace it with an RX888 and make few - if any - changes.  
  
As an example, let is consider the file "radiod@hf.conf" a piece at a time.  
  
ALL radiod configuration files MUST start with a "global" section - and consider this example:  
  
```
[global]  
overlap = 5  
blocktime = 20  
input = rx888-status.local  
samprate = 12000  
mode = usb  
status = hf.local  
fft-threads = 4

```  
  
This is the only section of a radiod configuration file that may be specific to the receiver hardware in that one might use different "overlap" or "blocktime" values with much lower sample rates.  Taking each line:  

- [global] - This is the required first section that tells radiod how to get the raw data from the receiver hardware.
- overlap - This parameter may be used only in the [global] section.  ka9q-radio uses a large FFT (Fast Fourier Transform) in the process of getting data from the receive hardware.

- Simply put, this FFT creates a mathematical equivalent of the entire receiver's spectrum in memory from which the radiod program can pluck out portions of it representing individual receivers' passbands.
- Part of this process requires that some of the "new" data includes a portion of the "old" data and the value of "5" here means that 1/5th (or 20%) of this data is "old" data.
- A value of 5 represents a reasonable balance between CPU utilization and filter "sharpness".  For example, an overlap of "2" would mean that 50% of our data is "old" - meaning that we must processes more than twice as much data and we could get sharper filters.
- For a fast computer - and particularly with receive hardware like an RTL-SDR or similar with lower sample rates (and less data) we might as well use a value like 2, but for the RX888 where we are handling massive amounts of data, a value of 5 may be more appropriate.
- Only certain values work for "overlap", so stick with 2 or 5 unless you feel adventurous.

- blocktime - This parameter may be used only in the [global] section.  The value of "20" represents 20 milliseconds - or one block every 50th of a second.  In our example using an RX888 running at 64.8 Msps, each "block" would be 1/50th of the number - or 1296000 samples.  The size of the FFT block is a bit larger than this since it would include the "overlap" from the previous FFT cycle.

- Larger values of "blocktime" offer sharper filtering and allow more tolerance to the vaguaries of CPU scheduling - but will add a bit to the latency of processing - and increase CPU utilization as the CPU time "cost" increases with the square root of the FFT block size.

- input - This parameter may be used only in the [global] section.  This is the multicast hostname of the stream containing the status/control data from our receive hardware:  You'll notice in the example above that in the "status" definition in the rx888 configuration file above.
- fft-threads - This parameter may be used only in the [global] section.   This is the number of CPU threads spawned by the FFT processes.  In this example, if you have a CPU with at least 4 cores, this will spread the FFT processing among 4 cores rather than the default of having 1 core do it all.

- The total amount of CPU power when splitting the task among multiple cores is slightly higher than having just 1 core do the job as a degree of overall efficiency is lost by doing so, but if a single core of a processor, itself, is not fast enough, this can help spread the load.
- Splitting the FFT task among several cores can also reduce latency somewhat since the job gets done more quickly.
- Be careful with this setting, making sure that its setting makes sense on the hardware that you are using!

- status - This parameter may be used only in the [global] section.   This specifies the name of the status/control multicast stream that may be used to monitor, tune, adjust, create or destroy additional receivers.  
    

The next several parameters are optional in the "global" section.  

- samprate - When placed in the [global] section, this sets the default sample rate for the sections that follow it:  This default can be overridden in the sections defining the receivers, below.
- mode - When placed in the [global] section, this sets the default mode for the sections that follow it:  This default can be overridden in the sections defining the receivers, below.
- mode-file - If you do not wish to use the "modes.conf" file (typically "~/ka9q-radio/modes.conf") you can specify a different one here.
- wisdom-file - The Fast Fourier Transform algorithm used in ka9q-radio uses a "wisdom" file which contains some pre-calculated data allowing greater efficiency and improving performance - and this optimization can vary depending on the hardware and FFT settings.  By default this file live in "/var/lib/ka9q-radio/wisdom", but this parameter may be included to use a different wisdom file. 

- For more information about the wisdom file see the file "docs/FFTW3.md" (also at:  [https://github.com/ka9q/ka9q-radio/blob/main/docs/FFTW3.md](https://github.com/ka9q/ka9q-radio/blob/main/docs/FFTW3.md) )  
    

There are a few other parameters discussed in the "docs/ka9q-radio.md" file that are either rarely used or not (yet) fully supported.  
  
After the "[global]" section  
  
One of the most powerful aspects of the way ka9q-radio works is that after the initial "work" on the FFT is done, extracting signal from the resulting data is almost "free"-  in terms of CPU power by comparison.  
  
What this means is that in our example of a 64.8 Msps sample rate, we have already ingested and partially processed every signal across the VLF, LF, MF and HF spectrum, and that it takes relatively little extra processing power to demodulate it in some way.  In other words, a modest Intel i5 or i7 processor, with an RX888 you can receive and demodulate hundreds of individual frequencies simultaneously.  
  
As noted elsewere on these pages, the resulting received data is conveyed using Multicast:  The more traditional way of having a separate audio channel (how would you use and define hundreds audio streams in ALSA/Pulsaudio/Pipewire?) simply doesn't apply here - and using Multicast means that the "one to many" model may be easily implemented.  Since multicast is UDP and connectionless, they can simply be "listened to" on the LAN by any number of clients without incurring any additional load whatsoever on the server.  
  
For example, if we were do demodulate, individually, every WSPR, FT-8, FT-4, SSTV, WWV and CHU frequency simultaneously and also have a 96 kHz chunk of the CW band for each of the HF bands for, say, a CW skimmer, all of that "audio" data would be available to any and all computers on the same LAN as the computer hosting the receiver hardware - and since Multicast is a "broadcast", more than one computer on the LAN could use that same data.  (Note that Multicast data cannot be easily conveyed across the Internet or wireless connections and some types of LANs bounded by routers/switches - but that's a topic for the network admins!)  
  
In the sections of the radiod .conf file after the [global] section we can define virtual receivers individually - or as groups of receivers of similar content.  Consider the following example from the same "radiod@hf.conf" file:  
  
```

[WWV]  
data = wwv-pcm.local  
mode = am  
freq = "2500k 5000k 10000k 15000k 20000k 25000k"  
```

Here we have defined six virtual receivers tuned to the WWV/H frequencies of 2.5, 5, 10, 15, 20 and 25 MHz and overriding the earlier default of "usb" in the [global] section, specifying the use of AM for this group of receivers.  All six of these receivers are conveyed on a single multicast stream called "wwv-pcm.local" as defined by the data parameter - so how does one specify how to extract each one?  Mentioned in the "rx888d.conf" file is the "ssrc" and here we use that parameter to specify the sub-stream carrying the individual frequency.  
  
By default, ka9q-radio will assign the ssrc as the digits within the frequency parameter.  For example, if we want to listen to the 10 MHz WWV audio, we would remove the "k" from the frequency and use an ssrc of 10000 to extract it from the multicast stream called "wwv-pcm.local":  See the example for the "pcmcat" and "pcmrecord" commands in "[KA9Q-Radio Command Overview](http://www.sdrutah.org/info/ka9q_radio_command_overview.html)" file for the use of the multicast name and ssrc.  
  
In above example we also see that we have selected mode "am", overriding the default in the [global] section.  If you wish to have a new "mode", this may be defined in "modes.conf" - see below.  
  
More on the "freq" parameter and how it defines the related ssrc  
  
In the example above we have defined the frequency in kHz, with 2.5 MHz being 2500 kHz, or "2500k".  It is also permissible to define it in Hz, as in "2500000" or as MHz, as in "2m5".  Also mentioned was the fact that the ssrc - which is needed to identify the sub-stream with a particular receiver - uses the string of characters - letters removed - to construct the ssrc - so one must be careful as the following examples illustrate, each correctly defining a 2.5 MHz receive frequency - but in diffrent ways:  

- 2500k - This represents 2500 kHz and an ssrc of "2500"
- 2500k000 - This represents 2500 kHz, but specifying it to the Hz as in "2500.000 kHz" - but the ssrc here is "2500000"
- 2m5 - This represents 2500 kHz as "2.5 MHz" and the ssrc is "25"
- 2m500000 - This represents 2500 khz as 2.500000 MHz" and the ssrc is "2500000" - which is the same as the SSRC from "2500k000".

What the above tells you is that you must be careful and consistent in the way you define frequencies so that your SSRC can be easily divined from them.  
  
Practically speaking, you would represent the frequencies in each group in the same way so you would never be likely to have name conflict:  You also would not specify two identical instances of the same receive frequency, anyway, since as a multicast stream, multiple users/computers could intercept it!  Finally, if you did need to specify another instance of WWV in another receiver groups - say, one to "hear" the audio and another to monitor signal strength - the ssrc would not overlap, anyway, as that "other" group would have a different multicast address!  
  
Comments:  
  

- You can re-use the same ssrc as long as it's done in a different section and thus on a different multicast stream.  For example, in the above WWV section example, we know that on the data stream "wwv-pcm.local" and an ssrc of 5000, we will get the 5 MHz WWV signal.  If we had a different section - say called "TIMESIGS" that also contained a definition of a 5 MHz WWV receiver also using "5000k" as the frequency and our data stream was called "timesigs-pcm.local", we could re-use the ssrc of "5000" since it would be in a different multicast stream altogether.

- It is possible to define hundreds of frequencies to be received simultaneously.  In some cases, you would simply put these in different sections:  One such section is the "WWV" portion - and if you wanted, say, frequencies for W1AW or the time station CHU, these groups of frequencies could go in individual sections with those names.  Note, however, that each section so-defined will have its own multicast stream and set of ssrc values.  
    

In some cases you might want a large number of frequencies in a single section - an example being wanting to create a section called "2MRPT" where you might specify the frequency of every 2 meter repeater output on the band and have all of the outputs of these virtual receivers conveyed by a single multicast stream. While it might be possible to enter all of these on a single "freq" line, the parser (in "libiniparser" library) actually does have a limit on the length of the line. (Question:  What is this limit?)  
  
To spread out a long list of frequencies onto multiple lines - which can also make it easier to manage and read - but having multiple lines using the "freq" heading is not  permitted.  A work-around exists:  There are ten "aliases" of the "freq" field and these are "freq0" through "freq9", allowing your long list of frequencies to be placed on multiple lines as if there was a single line.  Note:  You can actually have 11 "freq" lines as you can use "freq", then "freq0", "freq1" and so-on.  

  
  
The "default" or "prototype" receiver:  
  
At the very end of the "radiod@hf.conf" file we note the following definition:  

```
[HF MANUAL]  
data = hf-pcm.local  
freq = 0
```

When "freq" is zero, we have a special case:  This creates a "protype" receiver that can be summoned into existance when, via the "control" program, we specifiy an previously-unused ssrc.  Using the "control" program (see "[KA9Q-Radio Command Overview](http://www.sdrutah.org/info/ka9q_radio_command_overview.html)") we can configure this receiver to any frequency, mode or bandwidth covered by the SDR hardware.  
  
Optional parameters  
  
The above shows a minimal example of what can be defined, but there are other parameters that may be included to define a receiver group:  

- disable - If you have a section defined in your "radiod.conf" file that contains one or more receivers that you wish to disable - but don't want to delete, simply include the line "disable = yes" in the definition.

- ssrc - Above, we went on at some length about how the ssrc is constructed from the frequency, but if you wanted to specify an ssrc explicitly, you could do that here.

- Question:  Does this apply only to the "[default]" group, or a group with a single receiver - how might one apply it to multiple receivers.  While this is shown in the "rx888d.conf" file above, does it even apply elsewhere?

  

---

  
The "modes.conf" file  
  
Mentioned previously, one can arbitrarily define "modes" in ka9q-radio.  In this context, a mode can be any combination of:  

- Sample Rate.  Certain modes - like those used for FM - require a higher sample rate to convey the signal.  Similarly, a higher bandwidth might be desirable for certain digital modes, or to send a portion (e.g. 7050 kHz,+/- 40 kHz) of an HF band to a CW skimmer or similar.
- Bandpass Filtering.  For USB, one would have a bandwidth from 200 to 2700 Hz, but for LSB this would be -200 to -2700 Hz, the negative frequency indicating that the passband was below the center of the receive passband.  Similarly, different bandwidths are needed for FM modes - and for a hypothetical "mode" that is used to send a portion of an amateur band to a CW skimmer - and yet a different one that might be used for WSPR reception.
- Demodulation method.  In ka9q-radio two types of demodulation are broadly defined:  "Linear" and "FM".

- Linear modes are those that require/use an amplitude component to convey intelligence - and examples of this are SSB - which consists generally of an amplitude envelope and frequency components, AM, which is less about frequency and mostly about amplitude, and other related modes.  A special case of this is also "I/Q" mode where one conveys spectral information using two audio channels (in quadrature) from which frequency information may also be derived.  Digital modes - even those that have no amplitude information (e.g. WSPR, FT-8) are also conveyed linearly since the combination of a multitude of signals within an SSB passband require that all of this information being represented by a myriad of different carriers must be done with amplitude.
- FM modes do not convey their information by changing amplitude and intelligence is conveyed solely by variation of frequency/phase.  This applies to the "FM" (really PM) that amateurs use on the VHF and UHF bands and to FM broadcast.  What is common to FM is that there is going to be only one signal within the passband of the receiver and, by its nature, the linear (amplitude) component is removed during its demodulation.

- Different AGC configurations.  In most cases where "linear" demodulation is used, an AGC (Automatic Gain Control) is employed to automatically adjust the received signal power - which will consists of the desired signal and other signals/noise within the passband - at a constant level.  This is the mechanism that allows SSB or AM signals of widely disparate signal strengths to have the same apparent "loudness".

- In some cases - such as the analysis of propagation - you might NOT want to employ AGC.  If the AGC is not used, the resulting signal level will be directly proportional to its strength impinging on the antenna, and if the system is calibrated appropriately - or if there is an available reference signal level provided locally on the antenna system - it is possible to measure the absolute signal power.

- Note:  The AGC that we are discussing here is that applied to the individual demodulators being produced by "radiod".  Another "layer" of AGC can be present on receive hardware (e.g. the AirSpy, RTL-SDR when operated above 30 MHz or with an upconverter, with an SDRPlay receiver) that is used to prevent front-end overload - and if meaningful absolute signal level readings are required, you would have to either disable this AGC or take its effects into account when doing the signal power measurements.

There are, no doubt, other cases where one might wish to define a "mode" for a specific application - the limit being mostly that of your imagination.  
  
The "modes.conf" file contains a number of pre-defined modes, and we'll discuss of of these below.  
  
PM (Phase Modulation)  
  
As mentioned above, on amateur radio the mode that we call "FM" is really "PM" (Phase Modulation).  In short, PM may be generated on an FM transmitter by boosting the audio at a rate of 6 dB/octave - which is to say that a given amplitude of tone at 1 kHz which causes +/- 2 kHz of deviation will, with a 2 kHz tone at that same amplitude will cause +/- 4 kHz of deviation.  Doing this "boosting of highs" cleverly reduces one of the unfortunate aspects of "true" FM:  When a signal gets weaker, it's is the high audio frequencies that get noisy first.  By making the high frequency component of the audio "louder" during transmit means that as the signal gets weaker and the noise starts to encroach, those high frequencies - being "pre-boosted" - will survive above the noise level better with weaker signals.  
  
On the receive end, the highs are "un-boosted" by precisely the same amount as they were boosted on transmit.  By doign this we not only restore the frequency response to its original level ("PM" audio on a "true" FM receiver sounds very tinny and shrill) but as the signal gets weaker and the noise appears in the high frequency audio, this, too, is reduced - improving the apparent signal-noise of the signal.  
  
Within the modes.conf file we see the following entry for PM - which, again, is the mode that we use on VHF and UHF: 
```
[pm]  
demod = fm  
samprate = 24000  
low =  -8000  
high = +8000  
squelchtail = 0  
threshold-extend = yes ; PM assumes voice mode, so enable this  
```
  
Note:  De-emphasis is enabled by default in ka9q-radio - see the comments about this in the [fm] section, below  
  
Breaking this down:  

- [pm] - Here we define the name of our mode, used in our "radiod.conf" file.
- demod - We are using an "FM" demodulator rather than a "linear" one.
- samprate - On VHF and UHF, a receive channel using "FM" (+/- 5 kHz) is typically 14-16 kHz wide - and to convey a signal of that bandwidth using I/Q requires a sample rate that could do this.  Here, a 24 kHz sample rate means that we could accommodate a signal that around 20 kHz wide if we had to.
- low and high - This specifies the lower and upper edge of the filter for this receiver, respectively.  In this case we are specifying +/- 8000 Hz, so our total detection bandwidth is 16 kHz - which is about right for a 5 kHz deviation "FM" signals.
- squelchtail - A squelch is used to prevent the "blasting of noise" when a signal is not present.  Since we are in the digital domain, we can specify a squelch tail of almost whatever we like.  In this case, the parameter is in units of "blocktime" as defined in the [global] section of our "radiod.conf" file - which, in the example above, is 20 milliseconds.  In the example above, we can set this to "0" which means that there is no squelch tail when a signal drops off.
- threshold-extend - When receiving a weak signal, noise results due to growing uncertainty as to the instantaneous frequency of the FM carrier.

- It is possible, however, to rule out some "impossible" (or at least unlikely) mathematical instances where the noise causes what is clearly a "bogus" result - and throw those out.
- In this example, the value of "yes" means that we enable this feature which may subjectively improve the apparent quality of weak signals.

See the "comment" section at the bottom of the "NFM" section, below for a note about the setting for de-emphasis in "PM" mode.  
  
  
Now let us look at the "Narrow FM" entry in the modes.conf file:  


```
[nfm]  
demod = fm  
samprate = 24000  
low =  -6250  
high = +6250  
deemph-tc = 0  
deemph-gain = 0  
threshold-extend = no ; don't interfere with packet, digital, etc  
```
  
Let's take a look at the differences between [pm] and [nfm]:  

- [nfm] - Here we define the name of our mode, used in our "radiod.conf" file.
- The "demod" and "samprate" are the same as we are dealing with generally similar signals in terms of modulation and bandwidth.
- low and high - In this case, we are demodulated "narrow fm" which has a deviation of +/- 2.5 kHz - half of that in our example above - but don't believe for a second that it takes half the bandwidth!  Following "Carson's Rule" (Wikipedia article [here](https://en.wikipedia.org/wiki/Carson_bandwidth_rule)) we can estimate the required bandwidth which is set here to be +/- 6250 Hz, or a total of 12.5 kHz.
- deemph-tc - Normally set to a decimal value in microseconds, this is more appropriate for "WFM" (Wide FM) as used in broadcasting, at it sets the transition frequency below which the signal amplitude is "flat" and that above which it is pre-emphasized by 6dB/octave.  Since we are demodulating "true" FM, we would disable this entirely by setting it to zero.
- deemph-gain - When you de-emphasize a signal, you reduce the gain of the signals overall - particularly those at the high end and this value - in dB - can boost it to make it objectively louder to compensate.  In this case - particularly since we have disabled de-emphasis (above) we set it to zero.
- threshold-extend - Because it can distort the audio a bit, we are disabling this feature when we want to receive data - more on this below.

You might ask "If we don't use 'true' FM on amateur radio, why have this?".  This would be true for analog voice, but you would use "true" FM for many digital voice modes like D-Star, C4FM (Yaesu Fusion) and FSK (e.g. that used for 9600 baud packet) and similar which do NOT use pre-emphasis.  Because of the possiblity that the "threshold-extend" can slightly mess up the signal - something that we wouldn't notice when listening to voice - we turn it off to keep it as pristine as conditions allow.  
  
Comment:  

  
You might wonder why in "true FM" mode we explicitly disable the de-emphasis while in "PM" mode we don't specify anything at all?  As it turns out, in the ka9q-radio code there are default value set for "deemph-tc" and "deemph-gain" which are 530.5 microseconds (which translates to 300 Hz) and 12 dB, respectively.  
  
What this means is that in the [pm] definition, above, our de-emphasis begins at about 300 Hz, rolling off at 6 dB/octave above that frequency - and the "deemph-gain" is 12 dB to boost the audio to make it "sound" like the right loudness as the rolling-off by the de-emphasis necessarily reduces the amplitude of the upper audio frequencies.  Since most amateur transmitters filter out voice below 300 Hz, anyway, we won't really miss it - particularly because control tones like subaudible and DCS signals reside in the frequencies below 300 Hz and are typically filtered out on receive by modern radios.  
  
It may seem to be a bit confusing to have the default for "fm" actually being appropriate for "pm" - and it confused me initially, but it makes sense since "pm" would be more-commonly used for VHF/UHF amateur communications than "true FM".

  
Now let's look at a USB mode definition:  
```
[usb]  
demod = linear  
samprate = 12000  
low =    +50  
high = +3000  
```

``
- [usb] - Here we define the name of our mode, used in our "radiod.conf" file.
- demod - Since it isn't an FM or related mode, it will be linear.
- samprate - Here we use 12 ksps which is more than enough to handle our "high" filter of 3 kHz, below.
- low and high.  Here we specify a "low" end value of "+50 Hz" and a high value of "+3000 Hz" - both of these being the lower and upper edges of the band-pass filter above the center frequency - hence "upper" sideband.

- If we wanted LSB (lower-sideband instead) we might specify a "low" value of -3000 Hz and a "high" value of "-50" Hz - these defining the lower and upper edges of the filter below the center frequency.

  
Additional parameters in "modes.conf":  
  
In addition to the above, there are quite a few parameters that can be used to configure a "mode".  For examples of the use of some of these, please take a look at the "modes.conf" file.  In the section below, you will see some of the parameters that have already been discussed, offered here for completeness.  

- samprate - This defines the default sample rate for the mode, but be sure that the sample rate is adequate for the type of signal that you are receiving!  This can be overridden in the "radiod.conf" file if needed.
- demod - Above we demonstrated "fm" and "linear", but there is also the special case for "wfm" ("wide FM") which is used for FM broadcasting.

- WFM contains a multiplex decoder (for stereo) and is always set to 48 kHz sample rate.

- channels - This is always "1" for "FM" modes and 2 for "WFM" modes.  It can also be set for "2" for modes like I/Q which required two channels for the quadrature data.

- You can also specify "stereo" instead of "channels = 2" or "mono" instead of "channels = 1".
- If the envelope detector is enabled (discussed below) the linear output appears on the left channel and the envelope detection appears on the right channel.  
    

- squelch-open and squelch-close - The defaults for this are 8 and 7 dB SNR (Signal-Noise ratio), respectively - and since the "open" threshold is higher than the "close" threshold, this provides some hysteresis to reduce "flapping" that would otherwise occur with a weak signal.

- For FM, the SNR is estimated from the mean and variance of the signal amplitude (which for FM is, by definition, constant) meaning that a variation is due to noise.
- For AM, a PLL is used to detect the presence of a carrier - and this PLL is used even for envelope-detected AM (as opposed to synchronous AM).

- To disable this entirely, set the "squelch-open" threshold to something like "-1000" (e.g. -1000 dB).

- squelchtail - This  value is defined in terms of the number of "blocks" (defined in the [global] section of the "radiod.conf" file).  A typial value of this is 20 milliseconds.  Setting a value of zero means that the squelch is closed (e.g. audio is muted) with minimum possible delay after the signal has been determined to have disappeared, according to the "squelch-close" setting, above.  
    
- low - This sets the lower edge of the predetection filter passband.  This frequency is with respect to the center frequency of the receiver as the "usb" example, above demonstrates.  The "note" portion of in the explanation of the "high" parameter, below, applies here, too.
- high - This sets the upper edge of the predetection filter passband - the compliment of "low", above.

- Note:  The "predetection" bandwidth does not necessarily have anything to do with the post-detection audio passband response.  The most obvious example of this is for FM detection:  An amateur FM signal with, say, +/- 5 kHz deviation must have a receive bandwidth of about 16 kHz (see the "pm" example above) to receive it with acceptable distortion - and this is true even if the audio on that FM signal extends only 3 kHz or so.  This is because the act of frequency/phase modulating a signal creates a large number of sidebands that extend far above and below the center frequency and we need a wide enough bandwidth to receive enough of them to recreate the original signal with adequate fidelity.
- In the case of AM, the total bandwidth is important since all signals that fall into it are demodulated:  If all signal within the passband are related to the original AM signal, they will contributed to its fidelity - but if they are unrelated (e.g. noise, other signals) they can degrade it.  Since true AM is double-sideband, we could receive just one "half" of the signal (taking care to preserve the carrier - which we need for AM!) and filtering out the (redundant) half of the signal where interference might be.
- In the case of SSB, since we are simply converting a portion of the RF spectrum to audio, the "low" and "high" directly set the frequencies that are passed relative to the "dial" frequency (e.g. carrier).

- headroom - This is defaulted to -15 dBFS (dB below full-scale) and is used to set the target audio level when AGC is used.  In other words, when the AGC is active, it will always attempt to keep the audio at this level, no matter how strong/weak it is.
- shift - This defaults to 0 Hz and is applicable only to "linear" demodulation.   Typically used for the definition of a CW  mode (see "modes.conf" for an example) this shifts the signals above (positive number) or below (negative number) the center frequency.

- This shift is applied after downconversion, baseband filtering, and PLL tracking, if enabled.
- This is not the same is offseting the "low" and "high" filters parameters.
- Note:  This can be used to create a CW filter in the manner of many modern receivers as follows:

- On many modern receivers, when in CW mode, the frequency on the dial is NOT that of the "zero" Hz frequency of the receiver as it is SSB, but rather the dial shows the frequency of a CW signal that is placed at the center of its filter.

- For example, if your radio uses a CW offset of 700 Hz and you tune the CW signal to produce a 700 Hz audio tone, the dial frequency is the actual transmitted carrier frequency of that signal.

- Note that this is NOT the same as some older receivers (e.g. Drake, Heathkit) where they often used USB or LSB and the dial frequency showed where the "zero beat" frequency of the signal would be, requiring the operator to mentally offset the frequency display from the received tone.

- recovery-rate -  This is used for the AGC and applies to linear demodulation only.  This specifies the recovery rate - in dB/second - at which the gain is increased after a strong signal disappears and the hang-time (below) has passed since this happened.  The default is 20dB/second.
- hang-time  - This is used for the AGC and applies to linear demodulation only, specified in seconds.  This is the amount of time to hold the gain constant after the signal input has decreased.  An example of this being useful is on the reception of an SSB signal:  Without a "hang time" to keep the AGC constant for a short period after a signal peak, the audio would "pump" badly.  The default value is 1.1 seconds - which corresponds to a "slow" AGC found on most radios:  If you want a faster AGC (e.g. common for CW reception) this value would be reduced.
- threshold - This is used for the AGC and applies to linear demodulation only, spedified in dB.  This sets the threshold at which the AGC action will occur on noise without signal.  For example, if we set "headroom" to -15dBFS and use the default threshold value of -15 dB, the AGC will attempt to maintain the noise level at -30dBFS.
- gain - Linear demodulator only.  (Fixed values are forced in FM and WFM mode).  This sets the gain of the output of the linear demodulator before transission on the output stream.  If AGC is enabled, this sets the value before the AGC operates which can prevent a loud burst of noise (set too high) or taking too long to recover (set too low).  The default is 50 dB.

- If AGC is disabled, this sets the gain of the demodulated signal path with respect to the input signal.

- envelope - Linear demodulator only.  This sets the AM envelope detector.  If a 2-channel mode is selected, the envelope detector is sent to the right channel, but the linear detection appears on the left channel.  The default is "off".  
- pll - Linear demodulator only.  This  can enable the phase-locked loop, which will cause the receiver to try to track a carrier to keep it at zero Hz (center) - useful for synchronous AM demodulation.  Default is off ("pll = no").  Use  "pll = yes" to turn on.
- square - Linear demodulator only.  This enables the PLL with a squaring operation in the feedback path - used for synchrons detection of a DSB SC (Double-Sideband AM with suppressed carrier) signal.  If this is used, it also implies the use of the PLL (above).  Default is off ("no").
- pll-bw - Linear demodulator only.  This specifies the bandwidth of the PLL loop in Hz and it implies the use of the PLL.  The default is 100 Hz.
- agc - Linear demoduation only.  By default, this is on - but it can be used to disable the AGC entirely ("agc = 0").

- As discussed above, disabling the AGC is useful if you are trying to make measurements of absolute signal/noise power.
- Use the "gain" parameter to "manually" set the signal gain in the demodulator different from the default of 50 dB.

- deemph-tc - Not used for linear modes.  This sets the de-emphasis (in microseconds) for FM demodulation.

- Because of the defaults, below, "FM" always has de-emphasis enabled for PM reception even through "true" FM does not have de-empahsis.  
    

- For "FM" mode (e.g. amateur) the default is 530.5 uSec (300 Hz)
- For WFM mode this is 75 uSec - the value used in North America and South Korea:  Most other parts of the world use 50 uSec.

- deemph-gain - Not used for linear modes.  This sets the gain after de-empahsis and is chosen to achieve the appropriate "apparent loudness" as compared with the de-emphasis disabled.

- This is more important for "FM" mode as the 300 Hz "knee" frequency for de-emphasis (specified in "deemph-tc") is below the frequency of speech as passed by a typical amateur transmitter, making it sound "quiet".  The default for this in "FM" mode is 12 dB.
- In WFM mode, the default value is 0 dB as the "knee" for the de-emphasis is above the majority of the speech/music energy.