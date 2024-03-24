# RuxpinStream
A simple ESP32 generator of Teddy Ruxpin (and Grubby) automation streaming packets

DISCLAIMER: THIS INFORMATION IS PROVIDED FOR INFORMATIONAL USE AND WITHOUT ANY CLAIM OR WARRANTY OF ACCURACY. USE THIS PROGRAM, DESIGN AND INFORMATION AT YOUR OWN RISK.

This is a proof of concept demo for testing the encoding of the automation data stream for the Teddy Ruxpin. Teddy Ruxpin was introduced in 1985, manufactured by Worlds of Wonder. It used specially coded tapes to combine talking/singing with movements of eyes and upper and lower jaws. There is also provisions for another 3 channels of automation data for Grubby, a companion doll that had the same 3 channels of motion, an audio output circuit, but no tape deck. Grubby required a connection to Teddy to function.

The tape is recorded in stereo with the left tape channel being the audio and the right tape channel being the automation data stream. Teddy Ruxpin and Grubby are encoded with an 8-bit data stream. The channels are as follows:
Channel 1: unused
Channel 2: Teddy's eyes
Channel 3: Teddy's upper jaw
Channel 4: Teddy's lower jaw
Channel 5: Teddy/Grubby audio control
Channel 6: Grubby's eyes
Channel 7: Grubby's upper jaw
Channel 8: Grubby's lower jaw

Data is sent in packets or frames. Each packet contains data for all 8 channels. The data for each channel is encoded into the width of a "low" pulse, preceded and followed by a 400us "high" clock pulse. So each packet contains 9 clock pulses separated by 8 data pulses. The data pulse is expected to be between 630 and 1590us in length. The maximum packet length is (theoretically) 400us x 9 clock pulses (3600us) + 1590us x 8 data pulses = 16,320us/16.32ms. The minimum packet length is (theoretically) 400us x 9 clock pulses (3600us) + 630us x 8 data pulses = 8640us/8.64ms. The packets are separated by a "low" pulse of approximately 4-5ms.

These timing values came from https://github.com/furrtek/Svengali . Experimentation shows that the circuit in practice is fairly tolerant of variations in timing. The lower limit of the data pulse duration was 500us in the Teddy I currently have available. For purposes of compatibility, sticking to the "reference" values listed is probably good practice.

The data values for each channel are interpreted as follows:
Channel 2: minimum value = eyes fully open; maximum value = eyes fully closed
Channel 3 & 4: minimum value = jaw fully closed; maximum value = jaw fully open
(It is assumed that channel 6 function the same as channel 2 and channels 7-8 function the same as channels 3-4).
Channel 5: minimum value routes audio to Teddy; maximum value routes audio to Grubby (if connected)

This program uses the ESP32 Timer 1 in alarm mode to generate the pulse train timing. A simple state machine tracks which pulse is currently being output. Timer 1 runs at a frequency of 1MHz, 1/80 of the clock frequency. The timer is reset to begin counting up at the beginning of each pulse. When the timer value equals the alarm value, an interrupt is generated. The Interrupt Service Routine toggles the output pin, determines which channel's data (or clock pulse) is being sent and begins the next timing cycle. 

As this runs automatically in the background it requires no ongoing maintenance. Changing the vales of the data for each servo channel is effected by changing the corresponding array element in the servo[] array. This demo has no provision for changing channels 5-8, as I have no Grubby to test it. An additional switch could be added for channel 5 and the channel 6-8 data could just be copied from the data in channels 2-4, or another 3 potentiometers could be added to control the channels separately.

The data stream is fed into Teddy using a headphone-to-cassette adapter. It was used in the "olden days" to get audio from the headphone output of a portable CD/MP3 player into a stereo that has no line input, but has a cassette player. The 3.3V output of the ESP32 GPIO is attenuated through a 50K variable resistor to allow reducing to the right value to couple to the desired device. An output capacitor blocks DC through the output jack. That probably makes no difference for the headphone-to-cassette adapter, but it may if connected to the input of a cassette/digital recorder. The circuit wasn't really designed for connection to a recording device, as it's just a proof of concept.
