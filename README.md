# C64 Video Enhancement

FPGA based modification board for the Commodore 64 computer to produce YPbPr video output.

The C64 is the most iconic device of the 8-bit computer era and there is a huge library of
software and hardware modifications available.
Nevertheless the video output quality is inherently poor by modern standards, even with the use
of the s-video option. This is specially true with LCD flat screens that amplify the 
visual artifacts even more.

The biggest problem here is the fact, that the VIC-II video chip directly creates a
chrominance signal in the weird and strange way that is necessary for use with
analog television systems. It is then basically impossible to truly convert it back to 
any form of RGB or other component signal (and I tried really, really hard).
And even the luminance signal of the VIC-II is far from perfect and carries a lot of
noise from various sources.

So the way to go was then to bypass the chrominance/luminance signal generation of the VIC-II
and make a solution that computes a YPbPr signal directly from the digital information available
inside the computer. 
As it turns out, it is enough to passively listen to just about 22 pins of the VIC-II to figure out what 
video signal it actually intends to generate. Using the information from these pins and some
logic implemented in an FPGA, a pixel-perfect replication of the video image can be generated.

## Hardware

This mod evolved from a previous version that uses my generic A-Video board. It fixes the main
shortcomings of this version, as it was a very ugly install and it needed big holes in the case
for the three RCA plugs.
This board now is designed to replace the RF modulator and use the existing holes in the 
case. No need to modify the outside of the C64 in any way.

The mod set consists of two main parts:  
* VIC-II adapter
    This is installed between the existing VIC-II and its socket and sniffs all relevant
	signals and translates them to 3.3V logic levels for use by the FPGA.
* FPGA board
    This is connected via a ribbon cable to the adapter to receive the signal stream and
	provice the YPbPr output on a 4-pin TSSR jack. 
	Additionally this board carries the necessary electronics to take over the functions 
	of the removed RF modulator to amplify the composite and s-video signals (so the original A/V-jack 
	is still functional).
	
![alt text](doc/install_shortboard.jpg "Installation on a 'short' main board")	
	
## Compatibility

The mod is intended to eventually support all revisions of the C64: PAL and NTSC, 
long boards and short boards and 5V or 12V supply voltage.
The hardware is probably already compatible (tested with a long 12V-board and a short 5V-boards),
but for the various variants of the VIC-II (mainly NTSC) I still need to adapt the firmware. 
	
The mod was tested to work with the following variants of the VIC-II:
* 8565R2
* 6569R5

## Video output

The mod board generates a YPbPr signal which can be switched to one of three modes:
* 240p/288p progressive 50Hz/60Hz
* 480p/576p progressive 50Hz/60Hz using scanline doubling
* 480p/576p progressive 50Hz/60Hz with visual scanline effect

The signal is provided on a 4-pin TRRS-jack. The order of signals (tip to sleve) is as follows:  Y, Pb, Pr, GND.
This assignment seems to be a kind of standard, but you can use any breakout cable that converts TSSR to 3 RCA jacks, as long
as the common GND is located at the sleeve.

Use the three-state switch at the back to select the video mode. The 240p/288p mode is not supported by all TVs and will
probably create some de-interlacing artefacts. On the other side, this mode is perfect to feed into a dedicated
upscaler that can handle it (the famous 'Framemeister' or the 'OSSC' come to mind).

![alt text](doc/worldgames2.jpg "Output mode with visual scanline effect")	

## Installation instructions

* Remove the RF modulator
This requires some skill and a proper desoldering pump. I managed to do it with just a soldering iron and a manual pump
but I would not recommend it.
* Solder two 4-pin headers to the mainboard. These pins will later go into the FPGA board to provide power
and to carry the analog singnals in and out (to replicate the functionality of the RF modulator).
* Temporarily stick the FPGA board onto the pins and use it as a means to align the remaining pins correctly to the main board
(either two pins on each side if installed on a 'long' board or only two pins at the back if installed on a 'short' board) 
and solder them to the main board. These pins are needed to connect the GND as well as to provide the correct vertical spacing.
* Align the FPGA board vertically to the holes in the computer case and solder all pins.
* Remove the VIC-II and put the adapter into the socket. For some main boards (specifically ASSY 250407) this
additionally requires to first make same space at the right side of the socket by relocating a capacitor and a trimmer pot. 
Alternatively you could try to raise the adapter buy inserting an additional 40-pin IC socket.
* Connect the 20-pin ribbon cable to the FPGA board.


## Contact
If you want to receive a C64 Video enhancement board, feel free to contact me:
reinhard.grafl (at) aon.at  
I do not yet have a series production started, but I could produce a few sets manually
if demand is too low for a small series production.

