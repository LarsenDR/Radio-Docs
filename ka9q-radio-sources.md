
# Documentation of ka9q-radio

by Dave Larsen, KV0S


# Sources

This is a list of the iq radio sources that are currently used with the ka9q-radio software.



# rtl-sdr

#### References [osmocom.org](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr)

#### Hardware description

![rtl-sdr](/images/rtl-sdr-sm.png)
 
#### Resources

#### Drivers

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
---

#### Drivers

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

airspyhfd - a ka9q-radio driver program

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
---

#### References [funcube_dongle](http://www.funcubedongle.com/)


#### Hardware description

![funcube-dongle](/images/funcube-dongle-sm.png)

### Resources

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



**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico.

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://tapr.org/37th-arrl-and-tapr-digital-communications-conference/). 40th ARRL and TAPR Digital Communications Conference, Virtual On-Line.




