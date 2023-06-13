
# Overview and Module documentation of ka9q-radio

by Dave Larsen, KV0S

---


# Sources

This is a list of the iq radio sources that are currently used with the ka9q-radio software.

---

# rtl-sdr

#### References [osmocom.org](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr)

#### Hardware description

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/rtl-sdr.png rtl-sdr")
 
#### Resources

#### Drivers

#### Configuration

#### Outputs

---

# airspy

#### References [airspy.com](https://airspy.com/)

#### Hardware description

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/airspy.png "airspy")

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

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/airspyHF.png "airspyHF")

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

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/hackrf.png "hackrf")

#### Resources

#### Drivers

#### Configuration

#### Outputs


---

# funcube_dongle

#### References [funcube_dongle](http://www.funcubedongle.com/)


#### Hardware description

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/funcube-dongle.png "funcube-dongle")

#### Resources

#### Drivers

#### Configuration

#### Outputs



---

# RX888MKII

#### References [RX888](https://groups.io/g/NextGenSDRs)

#### Hardware description

![Alt text](https://github.com/LarsenDR/Radio-Docs/blob/main/rx-888-~MKII.png "rx-888 MKII")

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
# sockbuf

#### References

#### Hardware description

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


# References

---

**Karn, Phil**, KA9Q (karn@ka9q.net) 2018. [Realtime Multicast for SDR Module Interconnection](https://tapr.org/40th-annual-arrl-and-tapr-digital-communications-conference/). 37th ARRL and TAPR Digital Communications Conference, Albuquerque, New Mexico.

**Karn, Phil**, KA9Q (karn@ka9q.net) 2021. [The KA9Q-Radio Package](https://tapr.org/37th-arrl-and-tapr-digital-communications-conference/). 40th ARRL and TAPR Digital Communications Conference, Virtual On-Line.



