#Summary
This is a simple framework to perform a DPA analysis by using a SDR receiver.
It uses the Osmocom SDR source, so it is compatible with most SDR receivers such as RTL-SDR, Hackrf (rad1o) and USRP.
The mains focus was to attack laptops and desktop PC's.
It is required to start a small daemon on the DUT that allows (limited ;) remote code execution (see dut.py for details).
As most of this devices are leaking at a very low frequency ( < 4MHz) in most cases an additional upconverter is required.


# Installation and Requirements
Install the following packages:

    -GNURadio
    -OsmoSDR
    -python2
    -matplotlib, numpy

Compile grc/cap-demod/cap-demod.grc using GNURadio Companion

# Tools Overview
All programs expecting a configuration fgile as first argument.
See below for details.

## dut.py
This is the daemon running on the Device under Test.
It allows to start programs on the DUT over a Network connection and passing an argument to it.
The test programs are located in the cprog/ directory.

## capture.py
This tool can be used to calibrate the trigger and adapt the preprocessing e.g. modulation and digital filter.
It will trigger multiple executions and try to extract those.
This will guide you through the following steps:

    1. The raw trace containing 10 executions of the test program.
    2. The trigger signal. The executions should be clearly distinguishable to ensure a good succsess rate.
    3. Extracted trace for a single execution of one demodulated and preprocessed trace.

## dpa.py
Can be used to perform a DPA analysis for the specified program.
The arguments used for this are specified in dut.py for the test-program.
Intermediate results are stored in /tmp/out.

## rsa-crt.py
Attack implementation to perform a binary search on the RSA-CRT modul.
This works for openssl-exp\* programs by exploiting Side-Channel-Effects caused by the modular reduction of the Input.
To use less traces,  non relevant parts of the spectrogram should be masked.

## graph.py
Can be used to display .npy files generated by dpa.py

# Antenna
Best results were performed with a magnetic field probe as described by Genkint et. Al.
This consists of 3 turns of coax cable with a cut in the middle of the shielding.
At one end, the core is connected to the shielding and the shielding of the other end.
The signal can be measured between the open core and both connected shieldings.

![alt tag](https://raw.githubusercontent.com/bolek42/rsa-sdr/master/images/antenna.jpg)

On some systems fluctuations of the USB power supply can also be used for this attack.
A small capacitor should be used to protect the input of the SDR receiver.

![alt tag](https://raw.githubusercontent.com/bolek42/rsa-sdr/master/images/usb.jpg)

# Configuration
The configuration file is stored as JSON format. A sample file can be found at config/sample.json
All programs excepting a configuration file as first Argument.

##### misc
Some generic information about the DUT

Key | Description
--- | ---
cmd | Path to the test Program to run on DUT
ip | IP of the DUT
port | Port for the daemon to listen

##### capture
This holds the SDR configuration

Key | Description
--- | ---
frequency | Center frequency for Tuning the SDR
samp_rate | Capture sample rate for the SDR receiver
gain | Gain for the SDR receiver

##### trigger
Configuration for the trigger/alignment.
If dpa.py or rsa-crt.py hungs, this is most likely the place to search for misconfiguration

Key | Description
--- | ---
frequency | Absolute frequency for the carrier used for the trigger signal. This carrier should be stronger if the program is executed.
delay | Time delay between two executions of the test program. This can be used to speed up capturing, but can also destabilze the triggering process.
pre | Timespan in seconds to extract before the trigger point.
post | Timespan in seconds to extract after the trigger point.
low_pass | Cutoff frequency of the Input lowpass filter for the AM-demod.

##### demod
Key | Description
--- | ---
frequency | Absolute frequency of the carrier used for demodulation.
select | Triple of integers used to select configure the flowgraph. For details see below.
bandpass_low | Lowpass cutoff frequency used for Optional Bandpass filter
bandpass_high | Highpass cutoff frequency used for Optional Bandpass filter
decimation | Decimation rate after demodultion. A lowpass filter is used to reduce the Bandwith for the specified decimation rate.

To specify the type of demodulation and to apply a optional bandpass filter, the triple "select" with the following syntax is used:

First | Meaning
--- | ---
0 | No modification or decimation is applied to the captured signal.
1 | The input for AM-Demod of the trigger signal is selected.
2 | The input for the Demodulation is selected.
3 | The demodulated signal is selected.

Second | Meaning
--- | ---
0 | Frequency demodulation is used.
1 | Amplitude demodulation is used.

Third | Meaning
--- | ---
0 | Bandpass filter is not used
1 | Bandpass filter is used


##### preprocess
Key | Description
--- | ---
stft | Boolean to specify if a spectrogram should be used for DPA analysis.
stft_log | Boolean to specify if log10 shoudl be applied to the spectrogram.
fft_len | Number of bins used for the FFT (frequency resolution)
fft_step | Step width in samples for the spectrogram ( one FFT each fft_step samples)

Key | Description
--- | ---
mask | Boolean to specify if non relevant parts of the spectrogram should be masked with zero
mask_f1 | Symmetric bandpass masking high cutoff (0-1)
mask_f2 | Symmetric bandpass masking low cutoff
mask_t1 | Masking in time domain lower cutoff (0-1)
mask_t2 | Masking in time domain upper cutoff (0-1)
