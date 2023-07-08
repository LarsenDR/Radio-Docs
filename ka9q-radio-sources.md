
# Documentation of ka9q-radio

edit by Dave Larsen, KV0S


# Sources

This is a list of the iq radio sources that are currently used with the ka9q-radio software.



# rtl-sdr

#### References [osmocom.org](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr)

#### Hardware description

![rtl-sdr](/images/rtl-sdr-sm.png)
 
#### Resources

- Usage:    rtlsdrd [-A mcast_ifce] [-D mcast_dest_addr] [-I rtlsdr_dev_index] [-L] [-R metadata_dest] [-S rtp_ssrc] [-T rtp_ttl] [-a] [-b] [-c] [-f freq_hz] [-h] [-p IP_tos] [-r sample_rate] [-t mcast_stat_ttl] [-v]

- Note:  "rtlsdrd" does not (yet?) have an associated ".conf" file and all parameters are entered on the command line.

- The parameters and their defaults are:
- -v:  Verbose mode
- -A:  Multicast interface (e.g. eth0)
- -D:  Multicast destination address for RTP (PCM/audio/sample) data.
- -I:  Input device (Default is device "0")  This may allow the use of the serial number defined by the user using the "rtl_eeprom" utility.
- -L:  Set "linearity" using gain tables - default without this parameter is "sensitivity"
- -R:  Metadata destination address
- -S:  ssrc for multicast output data
- -T:  TTL (time to live) for RTP data
- -a:  Enable software AGC
- -b:  Enable bias voltage on antenna connector (e.g. "bias tee") - Approx. 4.7 volts at 100 mA, max.
- -c:  Frequency calibration.  This is a decimal number, positive/negative, to adjust the frequency.
- -f:  Initial center frequency in Hz
- -p:  IP TOS (Type of service)  (Default:  48  AF12<<2)
- -r:  Sample rate .  RTL-SDR dongles typically support sample rates of  225-300 kHz and 900-2048 kHz - the upper limit usually being limited by the USB hardware.  Sample rates of higher than 2048 may work in some cases (particularly on USB 3.0 hardware) but this would need to be tested on a per-device basis to determine if samples are dropped.
- -t:  Multicast status stream TTL (time to live)
- Note:  Clarification of "Linearity" and "Sensitivity" modes controlled by the "-L" parameter and how gain might be manually set is needed.  It is unknown how this module handles the use of "Converter" mode - via the R820T converter - and "Direct" mode as used <30 MHz.

This reads from an RTL-SDR dongle SDR,sending status and accepting control commands via its UDP socket.  

Comments:
- This utility currently lacks the following functionality regarding RTL-SDR dongles:
- The ability to select the "direct" ("Q") input that allows some RTL-SDR devices (such as the RTL-SDR Blog Version 3) to receive at HF frequencies.
- This ability to manually set the gain of the R820T frequency converter.
- It is useful to set the gain of the RTL-SDR manually in the following scenario:  In many metropolitan areas, there can be repeaters with signal levels that are >-50 dBm.  As the AGC action of the RTL-SDR will naturally set the gain to maximum in the presence of strong signals, when a signal greater than approximately -65 dB appears in the passband, the receiver will briefly overload prior to the AGC action occurring.  This causes two effects:
 - All signals being received will be briefly disrupted due to overload.  For voice monitoring, this causes an audible "click", but for AX.25 packet operation, this will probably cause the loss of the entire packet.
  -  Weaker signals will seem to appear and disappear depending on the current AGC-derived gain setting, the depth of gain reduction due to AGC action and the strength of the weak signals.  This issue is unavoidable due to the limited dynamic range of the RTL-SDR's 8 bit A/D, but a fixed gain setting would provide consistency.
  - It would also be useful to limit how high the AGC gain could go in the absence of other signals.
  - In practice, the manual gain could be set for the user's RF environment emperically - by listening to signals when the strongest local transmitters (e.g. repeaters) are active and/or observing the A/D's peak "dbFS" reading using the control program.

#### Drivers

librtlsdr-dev 

#### Configuration

```
[2m]
#rtl-sdr example conf file
#device = 0
calibration = -2.775e-6
iface = eth0
#hold-open = true
#tos = 48
#data-ttl = 0
#status-ttl = 1
#blocksize = 3840
description = 2m vertical
#ssrc = 0
status = 2m-antenna-status.local
data = 2m-antenna-data.local
```

#### Outputs


# airspy R2

#### References [airspy.com](https://airspy.com/)

#### Hardware description

![airspy](/images/airspy-sm.png)

 - Continuous 24 – 1700 MHz native RX range 
 - 3.5 dB NF between 42 and 1002 MHz
 - Maximum RF input of +10 dBm
 - Tracking RF filters
 - 35dBm IIP3 RF front end
 - 12bit ADC @ 20 MSPS (10.4 ENOB, 70dB SNR, 95dB SFDR)
 - 10MSPS IQ output
 - 80 MSPS when using custom firmware

#### Resources

**Usage:    airsply [-v] [-f config_file]**

The parameters and their defaults are:
 - -v:  Verbose mode
 - -f:  Config file.  (Default:  "/etc/radio/airspyd.conf")
If you are testing various configurations of airspyd and you are not running it as a service, it is recommended that you copy the original "airspyd.conf", rename it to something else (e.g. "airspyd_test.conf" and invoke that test file with the command line parameter of "-f airspyd_test.conf".
This reads from the Airspy SDR, sending status and accepting control commands via its UDP socket - can be run as a service.  Example needed.

The default configuration for airspyd may be found in the file airspyd.conf  - but note that the working version of this file will be found in /etc/radio  - so if you make any changes, you'll need to do so there, likely requiring sudo to edit it.  Also note that at the current time, if you reinstall/update ka9q from the .git, the configuration files in ~/ka9q-radio and /etc/radio will be overwritten.

See "airspy.conf" for configuration example below.

Parameters used in the ".conf" file with "airspyd":
- serial - Serial number of the Airspy device (default = null)
- amprate - Sample rate
- lna-agc - This enables/disables the AGC on the RF front end.  (default = 0)
- mixer-agc - This enables/disables the AGC associated with the mixer.  (default = 0)
- lna-gain -  This adjusts the RF front end gain (default = -1)
- mixer-gain - This sets the gain at the mixer (default = -1)
- vga-gain - This sets the gain of the variable-gain amplifier (default = -1)
- gainstep - (default = -1)
- bias - This enables/disables the bias voltage on the antenna connector (default = 0 for off)
- linearity - This sets the gain mode for linearity (default = 0 for off)
- agc-high-threshold - This sets the AGC high-level threshold (default = -10.0 dB)
- agc-low-threshold - This sets the AGC low-level threshold (default = -40.0 dB)
- frequency - The center frequency of the tuner in Hz (default = 0)
- name - The name of the device.  If unspecified, it constructed using the serial number.
- data-ttl - The data (RTP) multicast time-to-live (default = 0)
- status -ttl - The metatdata multicast time-to-live (default = 1)
- blocksize - The block size used for processing  (default = -1)
- description - This is a text description of the receiver - used in metadata.  This often describes the receiver, its purpose and/or antenna.  (default = null)
- ssrc - The ssrc of the RTP data.  (default selected automatically)
- tos - Type of IP service (default = 48  AF12<<2)
- data - The hostname of the RTP multicast stream
- status - The hostname of the metadata multicast stream
- iface - The default interface used for multicast data - typically "lo" (loopback) or "etho0".  (default = null)
- Note:  A default value of "-1" typically refers to a required parameter.

#### Drivers

libairspy-dev 

#### Configuration

```
# sample entry for airspy R2
[example]
description = "2m vertical"
serial = 91d064dc27839fcf

iface = eth0               ; force primary interface, avoid wifi
status = 2m-vertical.local
#status-ip = 239.1.1.1     ; default is hash of domain name
#status-ttl = 1            ; default

data = 2m-vertical-data.local
#data-ip = 239.1.1.2       ; default is hash of domain name
#data-ttl = 0              ; default
#blocksize = 960           ; default 960 when data-ttl > 0
#tos = 48                  ; IP ToS
#ssrc = 1234               ;default is based on time

# software agc (preferred)
#agc-high-threshold = -10   ;dBFS, default
#agc-low-threshold = -40    ;dBFS, default

# hardware settings
#sample-rate = 20000000    ; default is highest available
#bias = 0                  ; default no preamp
linearity = 1              ; default is off
#lna-agc = 0
#mixer-agc = 0
#lna-gain = 0
#mixer-gain = 0
#vga-gain = 5
#gainstep = 21             ; manual gain setting: 0-21, 21=maximum

#frequency = 148m8         ; locks tuner when manually set
```

#### Outputs

# airspyHF

#### References [airspyHF](https://airspy.com/airspy-hf-discovery/)

#### Hardware description


![airspyHF](/images/airspyHF-sm.png)

 - HF coverage between 9 kHz .. 31 MHz
 - VHF coverage between 60 .. 260 MHz
 - 18 bit Embedded Digital Down Converter (DDC)
 - Up to 660 kHz alias and image free output for 768 ksps IQ

#### Resources

#### Drivers

libairspyhf-dev 

#### Configuration
```
# sample entry for airspyHF+
[example2]
#samprate 912000           ; default
description = G5RV
serial = 3B52AA8090A12535  ; use airspy_info program from airspy.com to finded the serial number
iface = eth0               ; force primary interface, avoid wifi
status = g5rv.local
data = g5rv-data.local
hf-agc = 0
agc-thresh = 0
hf-att = 0;
hf-lna = 0
lib-dsp = 1
data-ttl = 0
status-ttl = 1
blocksize = 2048 # default for data-ttl = 0
#ssrc            # generated from time of day
tos = 48
frequency = 10000000  # default
```

#### Outputs

# hackrf

#### References [hackrf](https://greatscottgadgets.com/hackrf/one/)

#### Hardware description

![hackrf](/images/hackrf-sm.png)

### Resources

#### Drivers

libhackrf-dev

#### Configuration

#### Outputs

# funcube_dongle


#### References [funcube_dongle](http://www.funcubedongle.com/)


#### Hardware description

![funcube-dongle](/images/funcube-dongle-sm.png)

### Resources

- Usage:     funcubed [-v] [-f config_file] [-N name] [-L]

- The parameters and their defaults are:
- -v:  Verbose mode
- -f:   Config file (Default:  "funcubed.conf")
- -N:  Name
- This reads from the Funcube dongle SDR, accepting control commands from the UPD socket.  The default configuration is in "funcubed.conf" - Can be run as a service.  Example needed.

- Parameters used in the ".conf" file with "funcubed":
- device - This is a number that specifies the device, typically the serial number (default = 0)
- iface - The default interface used for multicast data - typically "lo" (loopback) or "etho0".  (default = null)
- data-ttl - The data (RTP) multicast time-to-live (default = 0)
- status -ttl - The metatdata multicast time-to-live (default = 1)
- status - The hostname of the metadata multicast stream (default = null)
- hold-open - This is a true/false value - not sure what it does (default = true)
- tos - Type of IP service (default = 48  AF12<<2)
- ssrc - The ssrc of the RTP data.  (default = 0)
- blocksize - The block size used for processing  (automatically determined)
- description - This is a text description of the receiver - used in metadata.  This often describes the receiver, its purpose and/or antenna.  (default = null)
- data - The hostname of the RTP multicast stream (default = null)
- calibration - Set frequency calibration
- bias - This enables/disables the bias voltage on the antenna connector (default = 0 for off)
- Note:  A default value of "-1" or "null" typically refers to a required parameter.

#### Drivers

#### Configuration

#### Outputs

# RX888MKII

#### References [RX888](https://groups.io/g/NextGenSDRs)

#### Hardware description

![rx-888-MKII](/images/rx-888-MKII-sm.png)

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

- Usage:    rx888d [-v] [-f config_file]

- The parameters and their defaults are:
- -v:  Verbose mode
- -f:  Configuration file (Default:  "/etc/radio/rx888d.conf")
- If you are testing various configurations of rx888d and you are not running i- t as a service, it is recommended that you copy the original "rx888d.conf", rename it to something else (e.g. "rx888d_test.conf" and invoke that test file with the command line parameter of "-f rx888d_test.conf".
- This reads from an RX-888 SDR, sending status and accepting control commands via its UDP socket - Can be run as a service.

The following parameters are used in the .conf file:
- firmware - This parameter specifies the name of the firmware file loaded into the RX-888.  There is no default, but the file "SDDC_FX3.img" is most commonly-used.
- queuedepth - (default = 16)
- reqsize - (default = 8)
- dither - This enables internal dithering of the A/D converter (default = 0)
- rand - This enables the "randomization" of the A/D converter's output (default = 0)
- att - The amount of attenuation in the receive signal path.  Range is 0 - 31.5 dB (default = 0)
- gainmode - This sets the gain mode as "low" or "high" (default = "high")
- gain - This sets the gain in the singal path.  Range is 0-34.0 dB (default = "1.5")
- samprate - This sets the A/D converter sample rate.  The minimum is 1000000 (1 Msps) - Useful maximum is around 130 Msps.  (default = 32000000)
- data-ttl - The data (RTP) multicast time-to-live (default = 0)
- status -ttl - The metatdata multicast time-to-live (default = 1)
- blocksize - The block size used for processing  (default selected automatically)
- description - This is a text description of the receiver - used in metadata.  This often describes the receiver, its purpose and/or antenna.  (default = null)
- ssrc - The ssrc of the RTP data.  (default selected automatically)
- tos - Type of IP service (default = 48  AF12<<2)
- data - The hostname of the RTP multicast stream (default = "rx888-pcm.local")
- status - The hostname of the metadata multicast stream (default = "rx888-status.local")
- iface - The default interface used for multicast data - typically "lo" (loopback) or "etho0".  (default = null)


#### Drivers

rx888d - a ka9q-radio driver program

Firmware - ka9q-radio/SDDC_FX3.img

#### Configuration

[g5rv]

```
# ka9q customized
description = "G5RV RX888"
firmware = SDDC_FX3.img
samprate = 64800000    ;  2^8 * 3^4 * 5^5
# needs fftw3 wisdom and/or fft-threads >= 4 and some buffer tuning
# seems to lose data in the network path
# forward FFT for 129,600,000 Hz, 20ms and overlap = 5 is 3240000
#samprate = 129600000    ;  2^9 * 3^4 * 5^5
iface = eth0               ; force primary interface, avoid wifi
status = rx888-status.local
data = rx888-pcm.local
ssrc = 10
;gain = 1.5 ; dB
gain = 10 ;dB - near floor of NF curve, still not too high for my G5RV
gainmode = high ; higher gain range
```


#### Outputs



# sockbuf

#### References

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs


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
 - 12bit 6.048MSPS – 8.064MSPS--- in field.

#### Drivers

#### Configuration

#### Outputs

# Horus

#### References

#### Hardware description

#### Resources

#### Drivers

#### Configuration

#### Outputs



# References


**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico. Video [Phil Karn at TAPR DCC 2018](https://youtu.be/D1LYLDGknOY)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://www.youtube.com/watch?v=VrMoNnctrqo&t=13s). 40th ARRL and TAPR Digital Communications Conference, [Virtual On-Line](https://youtu.be/kVY3E3e--_I?t=15080)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2022.  SDR Forum Dayton Hamvention. [PDF](https://files.tapr.org/meetings/DCC_2018/DCC2018-KA9Q-Multicast4SDR-Interconnect.pdf)

**Karn, Phil**, KA9Q (karn@ka9q.net) 2023.  [PAPA club meeting 2023](https://youtu.be/7nhBFSGby2o)




