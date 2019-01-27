# C64 Video Enhancement

FPGA based modificaton board for the Commodore 64 computer to produce YPbPr video output.

The C64 is the most iconic device of the 8-bit computer era and there is a huge library of
software and hardware modifications available.
Nevertheless the video output quality is inherently poor by modern standards, even with the use
of the S-Video option. This is specially true with LCD flat screens that amplify the 
visual artifacts even more.

The biggest problem here is the fact, that the VIC-II video chip directly creates a
chrominance signal in the weird and strange way it is necessary to for use with
analog television systems. It is then basically impossible to truly convert it back to 
any form of RGB or other component signal (and I tried really, really hard).
And even the luminance signal of the VIC-II is far from perfect and carries a lot of
noise from various sources.

So the way to go was then to bypass the chrominance/luminance signal generation of the VIC-II
and make a solution that computes a YPbPr signal directly from the digital information available
inside the computer. 
As it turns out, it is enough to passively listen to just about 22 pins of the VIC-II to figure out what 
video signal it actually intends to generate. Using the information from these pins and some
logic implemented in and FPGA, a pixel-perfect replication of the video image can be generated.

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
	compute a YPbPr output on a 4-pin TSSR jack. 
	Additionally this board carries the necessary electronics to take over the functions 
	of the RF modulator to amplify tge composite and s-video signals (so the original A/V-jack 
	is still functional).
	
## Compatibility

The mod is intended to eventually support all revisions of the C64: PAL and NTSC, 
long boards and short boards and 5V or 12V supply voltage.
The hardware is probably already compatible (tested with a long 12V-board and a short 5V-boards),
but for the various variants of the VIC-II (mainly NTSC) I need to adapt the firmware. 
	
The mod was tested to work with the following variants of the VIC-II:
* 8565R2
* 6569R5

## Video output

The mod board generates a YPbPr signal which can be switched to one of three formats:
* 240p/288p progressive 50Hz/60Hz
* 480p/576p progressive 50Hz/60Hz using scanline doubling
* 480p/576p progressive 50Hz/60Hz with visual scanline effect

## Contact
If you want to receive a C64 Video enhancement board, feel free to contact me:
reinhard.grafl (at) aon.at  
I do not yet have a series production started, but I could produce a few sets manually
if demand is too low for a small series production.

