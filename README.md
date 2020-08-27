# C64 Video Enhancement

FPGA based modification board for the Commodore 64 computer to produce YPbPr or RGB video output.

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

This mod evolved from a previous version that uses my generic 
[A-Video board](https://github.com/c0pperdragon/A-VideoBoard/tree/master/c64mod). 
It fixes the main
shortcomings of this version, as it was a very ugly install and it needed big holes in the case
for the three RCA plugs. It is compatible with the VIC-II adapter board and the firmware.
This board now is designed to replace the RF modulator and use the existing holes in the 
case. No need to modify the outside of the C64 in any way.

The mod set consists of two main parts:  
* VIC-II adapter
    This is installed between the existing VIC-II and its socket and sniffs all relevant
	signals and translates them to 3.3V logic levels for use by the FPGA.
* FPGA board
    This is connected via a ribbon cable to the adapter to receive the signal stream and
	provide a YPbPr output signal on a 4-pin TRRS jack. 
	Additionally this board carries the necessary electronics to take over the functions 
	of the removed RF modulator to amplify the composite and s-video signals (so the original A/V-jack 
	is still functional).
	
![alt text](doc/install_shortboard.jpg "Installation on a 'short' main board")	
	
## Compatibility

The mod is intended to eventually support all revisions of the C64: PAL and NTSC, 
long boards and short boards and 5V or 12V supply voltage.
The hardware is already compatible (tested with a long 12V-board and a short 5V-board).
but for some variants of the VIC-II (specifically the 6567R56A that was used in the very
first devices) you may need to update the firmware to the latest version.
The mod boards as sold by videogamesperfection.com have the firmware version 2.6 which already
works with most variants of the VIC. 

To support the 656765A chip, the mod additionally needs to be jumpered because it can not auto-detect
this specific variant on its own: You need to connect pin 5 of the JTAG header to ground. This can fairly easily be done by
connecting it to pin 2 of this header.


## Video output

The mod board generates a YPbPr signal which can be switched to one of three modes using the onboard slider switch
(right to left, when looking at the switch):
* 240p/288p progressive 60Hz/50Hz
* 480p/576p progressive 60Hz/50Hz using scanline doubling
* 480p/576p progressive 60Hz/50Hz with visual scanline effect

The signal is provided on a 4-pin TRRS-jack. The order of signals (tip to sleve) is as follows:  Y, Pb, Pr, GND.
This assignment seems to be a kind of standard, but you can use any breakout cable that converts TRRS to 3 RCA 
jacks, as long as the common GND is located at the sleeve.

Use the three-state switch at the back to select the video mode. The 240p/288p mode is not supported by all TVs and will
probably create some de-interlacing artefacts. On the other hand, this mode is perfect to feed into a dedicated
upscaler that can handle it (the famous 'Framemeister' or the 'OSSC' come to mind).
The 480p/576p modes are best used with TVs and give quite a good picture already without any additional upscaler.

![alt text](doc/worldgames2.jpg "Output mode with visual scanline effect")	

## Installation instructions

### Adapter board

* Remove the VIC-II and put the adapter board into the socket. The correct orientation for the
adapter is such that the ribbon cable either extends to the right or to the top (depending on your main board).
On some main boards, this could require to either relocate some 
passives or to insert an additional IC-socket to rise the adapter above these passives. 
* Put the VIC-II into the adapter. Make sure to put it in the original orientation.
* I have found that some boards have such a low-quality IC socket that it will not hold the adapter board tightly enough. 
If this is the case, you should replace it with a precision IC socket.
 
### Test the FPGA board (optional)

Before de-soldering the RF modulator, you can test the function of the main mod board. For this, temporarily connect 
one of GND3,GND4,GND5 somewhere to the C64 GND and the rightmost hole of RFCON2 to +5V. Connect the ribbon cable from the
adapter to the FPGA board. The TRRS plug should then already output a proper YPbPr signal.

### Install the FPGA board permanently

* Remove the RF modulator and remove all the solder from the pin holes.
* Install both 4-pin headers in the mainboard, pointing upwards. These pins will later go into the FPGA board to provide power
and to carry the analog signals in and out (to replicate the functionality of the RF modulator).
* Check into which holes in the FPGA board these pins will go. For installation on a "long" main board, these are the
8 holes at the edge, for a "short" main board, use the other set of holes.
* Temporarily stick the FPGA board onto the pins and use it as a means to align the remaining pins correctly to the main board
(either two pins on each side if installed on a 'long' board or only two pins at the back if installed on a 'short' board) 
and solder them to the main board, also pointing upwards. 
These pins are needed to connect the GND as well as to provide the correct vertical spacing.
* Align the FPGA board vertically to the holes in the computer case and solder all pins. For standard pins this
normally means to rise the FPGA board all the way up to the very tip of these pins. 
* If your VIC-II uses the higher voltage (all 6xxx - variants), you need to short the JPLUM1 solder bridge. 
* Connect the 20-pin ribbon cable to the FPGA board.

### Cabling

The 4-pin TRRS connector provides the YPbPr signal, and this can be connected to a TV or upscaler using different cabling options:
* A breakout adapter with a TRRS jack and 3 female RCA jacks. With this a standard YPbPr cable with male plugs can be attached.
* A cable with a TRRS jack and 3 male RCA plugs. This can be plugged directly into the YPbPr input on a TV.
* Some TVs also use a TRRS jack to input YPbPr, probably to safe cost and space. There it is possible to directly use a cable 
with two TRRS plugs on both sides (given that the TV uses the same pin assignment: Tip=Y, Ring 1=Pb, Ring 2=Pr, Sleeve=GND).


## Create your own kit
Everything to create the RF replacement board as well as the VIC adapter board (see the 
[A-Video board](https://github.com/c0pperdragon/A-VideoBoard/tree/master/c64mod)
repository) is completely open source and free to use for any purpose. 

Most parts are pretty standard, only the TRRS connector and the three-way switch are probably not that easy to source with
every electronics retailer.
For your convenience, I have set up a 
[BOM](doc/fpgaboard_bom.ods)
and a project parts list at 
[mouser](https://www.mouser.com/ProjectManager/ProjectDetail.aspx?AccessID=9f8979b031)
that contains everything besides the TRRS jack. For this jack you can use a "Cliff FC68125" with is available at 
several retailers or another part with the same footprint. 
For my original design I myself used an 
[unbranded part from a local retailer](https://www.conrad.at/de/p/conrad-components-klinken-steckverbinder-3-5-mm-buchse-einbau-horizontal-polzahl-4-stereo-schwarz-1-st-718732.html)
which did meet all my requirements and was pretty cheap as well.

Hint for assembling: The holes denoted GND1 - GND5 should not be populated with pin headers during assembly. It is intended that
pin headers that are installed into the C64 main board will stick up through the holes and are soldered as last step
of the installation.
Hint for testing: After the board is assembled and before installing it into the computer, you can already upload
a test pattern generator to the FPGA (available as binary release in the 
[A-Video project](https://github.com/c0pperdragon/A-VideoBoard/releases). 
Do this with a USB-Blaster connected to the JTAG headers. You also must provide 5V-9V on the power pin
(the rightmost hole of RFCON2 - as seen from the edge of the board) and 0V to any of GND3, GND4, GND5.  
The test pattern should show some fancy colors and three gradients to test each of the three signal components.
If the gradients look uneven or have less than 32 different steps each, you probably have some soldering 
problems in the resistor networks.

## Color palette
I have chosen very bright colors for the video output, to give the picture a fresher look. 
Especially "Mayhem in Monsterland" makes much more sense this way ;-)
I know that these saturated colors may not be to everybodies taste. If you want to reduce the color
saturation as a whole, you can replace the resistors R18 and R19 with lower values. 75 Ohm works fine, or
68 Ohm for even lower saturation. If you don't want to desolder these small parts, you could as
well solder an additional 150 Ohm right on top of each of the existing R18 and R19 (this also results
in 75 Ohm total). 

Beginning with firmware version 2.0, you have the possibility to freely choose a color palette that
suits your taste. It is even possible to reporgram the color signals to produce RGsB or RGB signals instead
of YPbPr. When you choose to disable sync, you need to get the sync signal from the original A/V port instead.
In this case it is even possible to build an adapter cable that joins the A/V signals and the RGB signals
together to a single SCART connector.

When you want to use a color palette according to the "colodore" algorithm, you can use Hojo-Norem's
[palette tuning program](https://github.com/Hojo-Norem/c0pperdragon-VIC-II-Palette-editor). 

## Details on color signal reprogramming
Beginning with firmware 2.0, the mod uses a set of 256 16-bit registers to hold the palette information.
These registers can be reprogrammed directly from the C64 and stored persistenly on the FPGA chip.
#### Unlock the registers
Before you can change the register contents, you need to write a value of 137 to memory address 53311.
Then switch the output mode switch to a different mode and back again (back is optional). This sequence is 
necessary to prevent untrusted code to modify the settings. This could a be a special concern on demo competitions.
#### Write into registers
For writing a 16-bit value, you need to first write the low byte into address 53308 and the high byte
into address 53309. Then write the target register number into address 53310. You can store the same value
into multiple registers by writing additional register numbers into 53310 without changing the data each time.
Note that it is not possible to read out any register contents. 
#### Store permanently
When you have changed the registers to your liking, you can store this permanently by writing the value 138
to address 53311. During storing this will also cause some random pixels to appear on the screen for a short time. 
#### Meaning of the registers
The registers can be thought of to form a 16x16 matrix (stored line by line), where each cell specifies the exact color
that needs to be produced when the computer wants to display one of its 16 colors, with the consideration
of which color has been displayed in the line directly above. 

If you do not want to simulate PAL color blending, all rows in the matrix will hold the same value. 
There it does not matter for the current line which color has been displayed above. 

If you want to do color blending, you can cook up any coloring blending 
scheme you want. The register accessed is always defined by currentcolor * 16 + colorabove. 

Each register contains 3 5-bit numbers that specifiy the analog signal levels to be sent on the signal outputs.

| Bits  | Signal   |
| ----- | -------- |
| 0-4   | Pr       |
| 5-9   | Pb       |
| 10-14 | Y        |
| 15    | unused   |

#### Disabling sync 
Bit 15 of register 0 has a special function, as it can be used to suppress the sync signal.
Writing a 1 into this bit will turn off sync on the Y line, whenever the 240p/288p output mode is selected.
In this mode, the luminousity line of the original A/V connector can be used to get the sync signal from
instead. 

## Contact
For technical questions and also to share your experience with this modification, please
use the issue tracking system of github. 

If you really insist on donating money, please feel free to send any amount to my paypal account: 
reinhard.grafl (at) aon.at
