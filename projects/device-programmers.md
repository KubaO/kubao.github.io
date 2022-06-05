---
layout: default
title: Projects
pseudo_parent: Home
nav_order: 2
permalink: /projects/device-programmers
---
{% include switcher.html %}

# Projects
{: .no_toc }

## Mostly To Do With Device Programmers
{: .no_toc }

The newest posts are on the bottom, none of this newest-first rubbish.
{: .fs-4 .fw-300 }

{% include toc.html %}

# On Device Programmers
*2021-01-23*

The state of affairs in the device programming department has been quite
lamentable lately.

## Modernity
How much would $500 buy you these days? A lot and a little, of course at once.
When buying new from vendors brands Conitec or Xeltek, expect to get support
for: a lot of modern memories - both parallel flash and serial memory, some
modern microcontrollers, ISP functionality, and a few PLD/CPLD devices thrown in
so that you don't feel too bad about the purchase. Expect no support for devices
made in the 1980s and earlier, save for a some EPROMs.  It's not all *that*
unreasonable, since there's little economic incentive to support what amounts to
"the vintage fringe".

Going thrifty, $150 gets you an XGPro T56, with the device support in the same
class as that of the $500 "mainstream" units. The software is not a pinnacle of
aesthetics, but it's certainly functional, and does what is says on the
(virtual) box.

Frankly said, what you get for $150 makes the $500 devices feel like a bit of a
ripoff.

## Antiquity

What about secondhand market? Other than occasional deals, $500 can get you
quite a bit, in fact. It is the price bracket for Data I/O 2900 programmers - a
featureful stand-alone programmers that are fully functional without any
software but that running on the programmer itself. It was a wise decision, and
made those models not too obsolescently cumbersome. Most importantly: you don't
need a DOS machine with a parallel port to run the programmer, as would be the
case for a majority of other "multi-device" programmers of that era.

Data I/O 2900 has *extensive* support for vintage devices, including pretty much
all PLDs from early 1990s going back to the very first PALs. It also supports
most bipolar PROMs, and 25V EPROMs. If you think of getting one of those, look
for ones that have semiconductor mass storage upgrade with "current" firmware
(V9.6), and are tested. There is some community following of those, so you won't
be totally alone if something breaks, but it is quite a complex product. If
something goes wrong you may end up spending quite a bit of time
reverse-engineering it to get it to work.

These programmers don't take generic package adapters - you need an adapter
designed for the 2900. The adapters contain PCB-level "type ID", implemented as
a few traces that strategically connect some of the signals together on the
programmer side of the interface. Thus far nobody has reverse engineered the
factory adapters to extract those type ID's, so you'd need to buy the adapters
on the secondhand market, and very few are available for purchase at any given
time. Of course, for devices that are available in both DIP and surface mount,
you can select the DIP device and use a generic adapter. For devices that only
came in SMT packages, the embedded software will insist on a compatible adapter.
The software runs *on the programmer* and you can connect to it using a serial
terminal.

# What's in the Box?
*2021-01-23*

A device programmer is a conceptually simple device, its primary function being
driving the pins of the programmed device according to a programming algorithm
and responding to the voltage and current feedback from the pins.

### Pin Drivers, Pin Drivers, Pin Drivers!
{: .no_toc }

Be proud, Mr. Ballmer, oh meme-worthy orator you!
{: .fs-4 .fw-300 }

Supplying the pins with current and monitoring them of the pins is done by,
well, pin drivers. At the most basic level, you can connect the GPIO pins of a
microcontroller directly to the signal pins of the Device Under Programming
(DUP), while supplying it with specified supply voltages. Those voltages may
need to be adjusted throughout the programming and verification process, e.g. a
5V part may need 6V supply while being programmed (e.g. some PALs), all the way
up to 25V (some legacy EPROMs). The "signals" on most memory devices remain in
the range between 0V and VCC, but this is almost never the case on vintage
fuse-programmed devices like PALs or bipolar PROMs: there, you often have to
drive the inputs with pulses that blow the fuses - 12V at a couple hundred mA is
not unusual.

Then we got the devices with more interesting character, where the pins have to
be driven with atypical slew rates, or even with exponential edges where the
voltage over time has to resemble the capacitor voltage charged from DC in an RC
circuit. Or the programming specification requires that the pins be pulled up or
pulled down by resistances within some range - never the same range, it seems.

In other words: things are conceptually simple, yet supporting most devices out
there calls for quite a bit of sophistication from the pin drivers. A pin driver
may need to act like a fast AWG, with current feedback used to drive some of the
state transitions in the programming algorithm - be it for fault detection or by
prescription from the vendor's programming algorithm. Or the pin driver must
emulate a resistance, and drive the current based on the voltage, in a closed
loop.

Thinking of what it takes to implement such a pin driver, one quickly realizes
that distributed "intelligence" of some sort is required. As the number of
device socket pins grows, it becomes infeasible to route all the pin driver
control and feedback signals to some central controller: it's way too many
signals. At a most basic level, the driver takes a target voltage and current
limit, and closes the control loop. Fast detection of current going above
threshold may be needed programming algorithm, or just by the programmer's
self-preservation instinct, a.k.a. overcurrent protection. It is an "instinct"
in the sense that the reaction to above-threshold current needs to be reflexive
and dependable. It the software is to be in the loop, it better be hard realtime
software.

# What Could Be in the Box?
*2021-01-23*

Device programming and device testing aren't far apart, even if we wanted to
limit ourselves to doing some limited parametric tests of the programmable
device before and after programming it. One may want to verify current
consumption at the supply voltage limits, verify the programmed contents and/or
the function of the programmed logic, do a JTAG boundary scan for devices that
support it, or even interact with the firmware on a microcontroller/SoC being
programmed, e.g. to calibrate its analog functions, or just verify
functionality.

## IC and Semiconductor Testing

Many programmers offer rudimentary device testing functions, usually limited to
74- and 4000 logic devices. Those are typically limited to detecting
overcurrents on supply and input pins, detecting open pins that shouldn't be
open, and driving a number of test vectors to the logic inputs and comparing
output logic levels.

In contrast, dedicated IC testers provide more extensive analog stimuli and
measurements, allowing testing of analog and mixed-function devices such as
op-amps, timers, and data converters (A/D and D/A). The cheap IC testers do have
very limited functionality, though, and it's hard to trust them to e.g. qualify
used parts. Retro-computing and service fields often have no choice but to
procure out-of-production parts on secondary markets, and the ability to test
the parts sufficiently to gain a modicum of trust in their reliability and
performance may be invaluable. Yet, even in the primary markets of in-production
parts there is the danger of fakes entering the supply chain. Their
functionality is often similar, e.g. a fake op-amp may still be functionally an
op-amp, but not the op-amp we paid for, and its specs will rarely match those of
the original device. Same goes for other parts, where e.g. rebadging efforts may
merely change the logic family labels, e.g. 74LS logic family may be sold under
the guise of a rarer variant like 74F. The tell-tales then would be the supply
current, logic signal input currents, output impedance at the logic outputs,
input thresholds and output voltage levels, as well as propagation times.

It is by now fairly obvious that even when programming and testing purely
digital devices, good analog testing functions come handy, as does the ability
to reliably capture lots of diagnostic data. This is where line between
semiconductor testers and device programmers becomes rather fuzzy, since there's
a large overlap in functionality: sufficiently flexible source-measure units can
be used equally well in both testing and in device programming.

## Small Board Functional Test

What else can be done with a "universal" device programmer/tester? The
mixed-signal data capture facilities should be already robust enough to
implement trustworthy testing and programming functions. Is there another use
for the flexible pin drivers and data capture? The idea of mobilizing our
programmer in the duty of small circuit board functional testing seems obvious
enough - the pin drivers suitable for device programming and IC testing can find
plenty of use in board testing, and for many simple designs the 48-96 probe
connections provided by the "programmer" may be enough. The market segment that
is price-sensitive enough to look for cheap solutions for board test/in-system
programming (ISP) is also most likely to wish to exploit the full value of the
tools they already have, and a device programmer that can double as a board
tester seems appealing if it lived up to its promise.

## Algorithm Reverse Engineering

There are other areas of pursuit that aren't often thought as primary
applications for device programmers, even though they perhaps should be. First,
if one had a fully open-source firmware and software for device programming,
it'd be (very sadly so!) somewhat unlikely to get official programming
specifications from the IC manufactuers. There are semi-legitimate reasons for
that, the most cited being the unwillingness to take on liability for failures
of parts mis-programmed by the "amateur" "hobbyist" device programmers. A
flexible-enough programmer could help out extensively in *reverse engineering*
of programming algorithms as implemented by other, "official" programmers. The
device with openly-undocumented algorithm can be probed in 1:1 fashion, and the
signals can be captured thanks to robust data capture capability we need anyway.
With a slightly more intelligent probe, each passthrough pin can be instrumented
not only for voltage measurement, but also for current measurement - if the pin
drivers are sufficiently sensitive and accurate, such measurement can be done
without any external active components. If we'd need to help overcome the limits
of our sniffer-programmer's voltage measurement, a few dozen fast high-side
current-sense amplifiers on the reverse-engineering probe seem like a reasonable
bargain.

## Algorithm Development

It's not unthinkable that a sufficiently dependable and flexible programming
system, designed from the ground-up to support user-defined algorithms, can find
use during programmable device development - if the R&D engineers have to
develop programming algorithms and processes, they may as well use an actual
programmer and not have to come up with a bespoke solution, and won't need to
re-purpose a functional tester with test development environment that may well
be unsuitable for quick iteration during chip R&D.

A question sneaks into the periphery: what's so special about user-defined
algorithms? Why are they not a common feature of higher-end device programmer
products, and on that note: what do the "professional" device programmer
developers use to implement algorithms? May it be that the environments they use
are rickety and only usable enough to support the core business and not much
else? User-defined algorithms don't deserve special treatment: all algorithms on
a flexible device programmer should be equally privileged, with same scrutiny
available on all three sides: the device programmer vendor's, the user's, and
the programmable device manufactuer's.

## Process Documentation

The device programmer market is somewhat uncompetitive, in part because the
semiconductor manufacturers usually only provide programming data under and NDA
and are leery of disseminating it outside of a select circle of "trusted"
programmer vendors. Under such secrecy, the device programmer can't really offer
sufficiently advanced diagnostic capabilities that would allow both a skilled
user (or a competitor!) to figure out the programming algorithm without leaving
the programmer's software suite. The end-user is just as disadvantaged as the
chip manufacturer: since no detailed data can be captured to document the
programming process, if there's any malfunction of the IC, an expensive and
tempestous finger-pointing battle unfolds between the IC manufacturer, the end
user, and the device programmer manufacturer. Even if everything is going
smoothly and amicable on the user's side, some higher reliability circumstances
may benefit from a sufficiently detailed capture of the voltage and current
signals present during every programming operation.

The benefits of data-capture-as-a-key-feature seem to be ever plentiful the
farther we look, and can bring lots of added value to the user even in
applications outside of the device programming area.

# Driving The Pins
*2021-01-23*

By now we have some idea of what's needed of pin drivers: they need to be
controllable voltage and current sources, potentially emulating virtual
impedances (e.g. a "resistor" pull-up/pull-down), with current and voltage
capacity sufficient to blow nichrome fuses, and fast enough to test and program
modern, high-capacity memory devices. On the other hand, it'd still be a hobby
project for me, and it's not feasible to throw 1Gs/s ADCs and DACs per pin -
even if suitably expensive FPGAs could deal with the torrent of data.

Gee whiz, I wonder if someone made pin driver chips available for the wider
public? Yes - at least Analog Devices offers a line of *fast* (up to 1.2GHz
driver speed) pin electronics/pin drivers: the [ADATE family][ADATE].

[ADATE]: https://www.analog.com/en/products/amplifiers/specialty-amplifiers/pin-drivers.html

# Teardowns
*2021-01-23*



## XGPro T56



## Conitec GALEP-5