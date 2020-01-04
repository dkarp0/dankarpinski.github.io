---
layout: post
title:  "Things I learnt building a Nixie Tube Clock"
date:   2020-01-04 15:57:27 +0000
categories: tech
---
Earlier this year I build this clock/Spotify player using old parts, new parts and a lot of trial and error. My background is in software engineering so electronics was entirely new to me. Here are some things I unexpectedly learnt along the way.

![Nixie Tube radio clock numbers showing 22:40](/assets/nixie-clock/clock-numbers.png)

### What wired radio is
I'd seen pictures of steam punk nixie tube clocks and that was the style I wanted to recreate. I found the following listing on eBay and a week later it was shipped in from Ukraine.
![Leningradets 1950s radio eBay listing](/assets/nixie-clock/radio-ebay-listing.png)
The item was listed as a "radio", but I didn't find any transistors or valves inside when I opened her up. Instead there was a big old transformer connected to a big old speaker. A few wires ran from different points on a basic rotary switch to different points on the transformer, adjusting the output voltage, which adjusted the volume. The power cable ran straight into this transformer.

With a bit of research I found out about [wired radio](https://www.nytimes.com/2001/10/18/world/moscow-journal-wired-radio-offers-fraying-link-to-russian-past.html): As a cheaper and simpler alternative to radio transmitters and receivers, the USSR sent their radio station by wire direct to homes. Homes were required to have a "wired radio" socket which carries the signal stepped up to 30V from the studio to increase range. The radio then steps the voltage down and sends it straight to the speaker.

### What Nixie tubes are and how to power them
Nixie tubes are cold-cathode tubes, most similar to neon lamps in how function. They give off a warm orange glow around a cathode. I picked up this set of 4 that had been ripped out of some old equipment in belarus:
![4 Nixie tubes eBay listing](/assets/nixie-clock/nixie-tubes-ebay-listing.png)

Unfortunately they need a ~180V power supply to run, which is a fair bit more than the 5V pins on the raspberry pi I wanted to use with them. Reading [some](https://threeneurons.wordpress.com/nixie-power-supply/) [articles](https://0x7d.com/2017/nixie-tube-clock/#HV_Power_Supply), I learnt that power supply is a huge topic on its own. Not willing to invest the time to do power supply safely, I bought a 20V - 180V nixie tube power supply off alibaba.

### How to control Nixie Tubes
The Nixie tubes I bought each have 11 evenly spaced wires pointing out the bottom. One of those is marked by a white blob, the anode. The rest are all cathodes and correspond to a number. Ground a cathode and the number is displayed, ground all the cathodes and a blur of all the numbers are displayed. With 4 nixie tubes, that's 40 cathodes that have to be switched to display the time correctly.

The cathodes have high voltage flowing through them, so you need a high voltage switch. The way these were originally controlled is using a chip like a [DM7441A/K155ID1](https://tubehobby.com/datasheets/k155id1.pdf). These take a BCD (Binary Coded Decimal) signal from 4 inputs (to give 16 bits of data) and switch 10 outputs (one for each decimal value). Sometimes the over-range (the 6 bits over 10) can also be used to disable the outputs, turning off the nixie tube. I purchased the following for my tubes:
![k155id1 nixie tube controller integrated circuit eBay listing](/assets/nixie-clock/k155id1-ebay-listing.png)

The 4 controller inputs can go straight to a raspberry pi's GPIO (general purpose input output) pins to be controlled using 5V. With 4 nixie controllers, we'd still need 16 (4x4) GPIO pins on the raspberry pi to controller the 40 cathodes on the tubes. An improvement, but still too many. Luckily I then found out about shift registers.

### What a shift register is
A shift register is a simple integrated circuit that lets you feed in inputs one after another (in serial), flip a switch and have all of those inputs outputted at the same time to another component. This means with a single Serial-in, Parallel-out (SIPO) shift register with just 3 input pins (data, latch and clock), all 16 nixie controller pins can be controlled. I learnt the basics of these from [sparkfun](https://learn.sparkfun.com/tutorials/shift-registers/all).

### The joy of designing PCBs
As the number of components increased, I had a lot of wires going everywhere and it was never going to fit neatly in the radio housing I'd bought. So I downloaded [KiCad](https://kicad-pcb.org/) and dove into the open source schematic editor and PCB layout software.

I found the learning curve pretty steep without having any background in electronics but, with the help of youtube and [reddit.com/r/PrintedCircuitBoard](https://www.reddit.com/r/PrintedCircuitBoard/), I was able to get designs ready for manufacture within a week or so. I spent a long time laying the traces beautifully around all the nixie cathodes into the controllers and registers over to input connectors. It was easy to get hooked refining where everything is placed over and over again.

![3 PCBs listed from JLCPCB](/assets/nixie-clock/pcbs-jlcpcb.png)

By staying within 100mm x 100mm, [JLCPCB](https://jlcpcb.com/) have a $1.99 deal for 5 PCBs. With express shipping for about $15, I received my boards within a week. Receiving your designs beautifully printed and shiny was a bit like christmas for me. Unfortunately, although KiCad has built in features to check your designs, there are plenty of things that can slip through. Which leads me to...

### The frustration of designing PCBs
It took me 3 iterations before I reached a PCB design that worked and I was happy with. I could still do another version and there are things I would change but I feel like it may always be like that. Version 1 had the wrong footprints which meant through hole components had to be soldered in spaces for surface mounted components and other components simply didn't fit. It also had no test points! Version 2 had a trace that wired a shutdown pin to 5v (which shut it down) where it should have been to ground. But version 3 finally worked!

### The importance of decoupling capacitors
While researching PCB design and reading data-sheets, I learnt that small capacitors should be placed near every integrated circuit. My understanding was that as the integrated circuits are switching the circuits, the length of the circuit can change and therefore the resistance of the circuit can change. This change in resistance can affect the voltage, which when unstable can then affect the behaviour of the integrated circuits. To stabilise the voltage, these capacitors are added that dump power back in when voltage drops. Placing them as close to the integrated circuits as possible makes them most affective.

While soldering the boards, I wanted to test them before committing to fully assembling them as I only had the 4 nixie tubes and it was some work to solder and de-solder them. So I soldered the minimal amount of components possible initially, skipping the decoupling capacitors. I noticed that the shift registers often behaved unexpectedly, perhaps because the raspberry pi also needed a better power supply. Adding these decoupling capacitors eliminated the issue.

### The importance of good ventilation
I spent a lot of time soldering (and de-soldering) over the 3 board iterations and used my living room coffee table for the purpose. After a long night inhaling fumes from the soldering flux, I noticed I felt quite unwell. Turns out this is not recommended and I later used a small fan and kept the window open.

![Nixie Tube radio clock](/assets/nixie-clock/clock.png)

### What PWM is
If you've ever seen LEDs or digital displays recorded through your phone camera, then you may have noticed that they often flicker. That is Pulse Width Modulation (PWM). Since you cannot easily dim an LED like you would an incandescent light bulb by reducing the voltage through it, LEDs use PWM to dim. The idea is to flash the light off and on thousands of times a second. Your eyes then average the light so you'll see it dimmed. The more it is off, the dimmer it is. The more it is on, the brighter it is. This is also the easiest way to dim a nixie tube.

### That PWM in python doesn't work
I naively tried a python loop that turned the nixies off and on many times a second. This does not work. The raspberry pi had to dedicate 100% cpu to the task. Even then, it was barely managing a hundred refreshes a second. The python interpreter simply has many other things to do, namely checking for the Global Interpreter Lock (GIL) periodically. You could actually watch the refresh rate go up and down as the system wandered between focusing on refreshing and doing other things.

### That PWM in code doesn't work on linux
Even outside of python on a raspberry pi running linux, you cannot get PWM to work just by coding in a loop in your code. The operating system has hundred or even thousands of processes to worry about and only a fixed few cores to worry with. Although I did rewrite the code in Golang and try anyway. It was significantly faster, but still didn't maintain a fixed rate. I did experiment with giving the clock process exclusive access to a cpu core, but this was clearly overkill. It shouldn't take a whole cpu core running at 100% to run a clock process.

And so I learnt why chips often have special clocks purely for providing PWM. You simply need to tell the system the frequency and how much it should be on (duty cycle) and the clock will generate a near perfect PWM signal for you using no cpu time at all. The raspberry pi has 2 channels dedicated to this purpose.

### TLDR
Radios that run down wires, old binary to decimal converters, shift registers, hours of circuit design, circuit boards in the bin, nausea, pulse width modulation and operating system schedulers can all be encountered while building a nixie clock.