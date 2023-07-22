[![GitHub license](https://img.shields.io/github/license/cartheur/hp5480-controller)](https://github.com/cartheur/hp5480-controller/blob/main/LICENSE)

# hp5480-controller (and crew)
A project for a controller to advance statistical analysis by a Hewlett-Packard 5480 DSO

# 141S analyzer restoration
A storage spectrum analyzer compliments the laboratory nicely. Learning about strict-hardware implementations of measurement. [Here -> 141S.](/141S/README.md)

# 3721A correlator repair
Advancing into statistical signal analysis. Worked (mostly) upon arrival but low-power board has issues. [Here -> 3721A.](/3721A/README.md)

# 8414B polar display repair
The 8407A with the 8412A Phase-Amplitude Display works very well. Now to add the ability to see impedence behaviour via polar coordinates. [Here -> 8414B.](/8414B/README.md)

## Background

This instrument was manufactured by Hewlett-Packard in the years 1968 - 1974.

![hp5480-view](/media/view.jpg "The instrument in operation")

More details forthcoming. Soon. Enough. Certainly.

## The controller

In order to test and diagnose the capabilities of the instrument, it is necessary to construct a controller from a raspberry-pi and a plug-board. The parts required are obtainable from Mouser.

![hp5480-view](/media/controller.jpg "The instrument in operation")

with four ICs connected to pins on the back of the instrument.

![hp5480-view](/media/connections.jpg "The instrument in operation")

An explanation of the code and how it is used is the following:

	ctl5480/	: Remote-control program for the 5480.
			  Interactive terminal/line-at-a-time program to be run from a unix shell.
			  C language. Make will produce a binary 'ctl5480'.

	util/		: Some utility routines needed by the ctl5480. compile/make this directory before ctl5480.

	HP5480Schematic.pdf	: Functional schematic of the system. This is mostly a re-presentation of HP's schematics.
				  This was done largely on a 'need' basis, there are some things left out: IC lists, physical locations, some boards of the 2-input analog plug-in.
				  HP's schematics are presented on a physical-module basis, they are redrawn to better present functional-module association.
				  The 4-input analog plug-in is presented, this uses some of the boards of the 2-input plug-in (ADC and 1 or 2 others) but not all.

	HP5480Modules.pdf	: Schematics of functional modules referenced in HP5480Schematic.pdf

	HP5480Timing.pdf	: Timing diagrams for the main timing system and some details for the Prepare state.

	HP5480Adapter.pdf	: Schematic of adapter to interface between the 5480 and controller-host (RPi or other suitable).

My redrawing of the HP schematics may or may not be useful. I did that while deriving some functional sense of the design in the absence of the Theory of Operation manual. If you're going between the HP schematics and my schematics you may find the renaming of symbol names annoying, but that, again, was to bring some functional clarity to things. I eventually arrived at breaking the internal system control down to a hierarchy of:

	1. function selection
	2. function execution control
	3. sweep control
	4. sample processing
	5. process micro-cycles
    .. and named things to better reflect that.

clt5480 has a limited help: "?" will get you a list of commands/arguments. It is focussed on exercising the accumulator and core memory as that's what I needed to do. I did the clocked-serial style interface just to minimize the number of IO pins to the controller-host.

In using ctl5480, or implementing some alternative remote-control, the major point to be aware of is that to do many/most things from remote control the timing system of the 5480 must be shut down. Otherwise, the memory continues to be scanned, and the address registers modified 'behind your back', as you try to do anything with memory via remote.
The timing system is shut down from remote by asserting nSVQ_MAIN and nSVQ_SUB (HP5480Schematic.pdf.4) (nMBSL asserted holds the timing system in reset).
I simplified this a tad in my unit with the internal mod (in red) so one only has to assert nSVQ_SUB, but the mod is not a requisite (actually, I just ran out of male pins for the rear-panel connectors).

Regarding the rear-panel connectors, I had a bunch of the correct male pins for those connectors but not the base & shroud. There were some on ebay but 50$+ (at that time).
A possibility was using short stubs of common #14 solid copper wire. #14 is slightly larger diameter than the proper pins but will fit. The downside was sharp edges of cut #14 might scratch the gold plating in the female pins and whether the larger diameter is unduly bending the spring metal in the F pins.