SPI Terminal Interim Report (28-Jun-2025)
========================================

This is the interim report on the SPI Terminal (SPI Term) project that was
funded during the 2024 Hacker Initiative Grant cycle. Principle investigator
is Chuck McManis. It is broken up into four sections, Design Progress, Software
Progress, Hardware Progress, and Next Steps. Feel free to reach out if you have
questions or something in this report is unclear.

## Design Progress:


The design process is focused on defining the requirements for the underlying
support hardware, and the register definitions for the resulting device. This
is done in the form of "threshold" type requirements where the minimums are
defined and a path for creating more capable devices is charted out so that
people building new SPI Terms from a new design can be compatible with the
existing ecosystem.

### Hardware Design

The hardware interface between SPI Term and the client is one of IIC, SPI, or
asynchronous serial at logic levels. SPI and IIC are self clocked, asynchronous
serial requires a training step first to identify the baud rate.

The hardware interface between the SPI Term and display devices can be one
of SPI, IIC, parallel bits, DSI, and DVI/HDMI. The hardware interface between
the SPI Term and input controls (keys, knobs) will be client defined by
avaialble resources in each particular SPI Term implementation.

Given the potential mismatch between the client and the rate at which the
SPI Term can respond to requests, the SPI Term presents a logic level 
flow control line IDLE/BUSY\*. Additionally, as the SPI Term can generate data
to be sent to the client it presents a logic level data avalable line DAV.
If left floating, their defaults are 'not busy' and 'data is available.'
That allows the client to send data (it will be silently discarded if the client
overruns the SPI Term), and to poll for available data. The design philosophy
here is to still be functional although with some decrease in fidelity that can
be compensated for in the CLIENT software stack.

All three interface modalities shall be presented by every SPI Term device. 

SPI Terms with a dedicated display, need only support that display. A SPI Term
that supports a user adding their own display should strive to support multiple
modalities for the display. 

Input controls for the SPI term consists of 'keys' and 'knobs'. Keys are
momentary contact push buttons which may be arranged in a variey of ways from
keyboards to gamepads. Knobs are rotating inputs that return a rotation value
that is sent whenever it is changed. Rotation values increase when the knob
is rotated clockwise, and decrease when the knob is rotated counter clockwise.

Indicator lights are supported as output and shall consist of serially
addressable RGB LEDs.  

### Protocol Design

The communication protocol with the SPI Term is designed to be similar to
protocols that talk to IIC or SPI devices, specifically a 'command' byte
with a R/W bit, and a register address. Followed by a protocol specific
number of 8 bit octets that can be read or written. Initiation of a command
byte is done through a protocol specific form, for IIC this is a START
condition, for SPI his is the assertion of chip select (CS), and for
asynchronous serial communications is it by passing three NUL characters (0)
in sequence. 

Registers are numbered from 0 and can range up to 16383. Values contained in
the registers read-only, write-only, and read-write. Attempting to write a
read-only value has no effect and reading an undefined value shall return
0. 

A typical transaction is then:

```
<start>[[R/W][E]Register][D0][D1]...[Dn]
```

The register value can be one or two octets. In the first octet:
  * Bit 7 determines if the transaction is a read (0), or write (1)
  * Bit 6 determines if the register number is short (0), or extended(1)
  * Bits 5 - 0 are the low order bits of the register.

If the extended bit is set, the second octet is the upper 8 bits of the
register number. If the extended bit is not set, the second octet is the
first data byte.

Register 0 is the status register and defines two octets of data

Register 1 is the ID register and always returns four bytes of data 
with defined as 0x52, 0x53, \<major\>, \<minor\>. 

One goal of this protocol was to enable polled operation where a read of
the first octet of register 0 would return sufficient state to emulate the
function of the data available and idle/busy hardware lines in software.

One of the downsides of the protocol is that asynchronous serial is more
packetized rather than a typical ANSI/ASCII terminal where you access
special functions with escape sequences. I plan to explore having a mode
that works that way understanding that getting into, and out of, that mode
would make for some challenges.

## Software Progress:

### Display coding

I've previously written drivers for a number of different displays, and am
using the code I wrote for the STM32F469 as a test bed. The STM32F469i-DISCO
has a display and a set of "Arduino compatible" pins (so it would be compatible
with Arduino shields. I'm using the digital I/Os to implement a simple scanned
keyboard type interface.

Additionally I'm using a 'Black Pill' (low cost STM32F401 dev board) attached
to an [AdaFruit 2.4"](https://www.adafruit.com/product/2478) display as a
lower res color screen. Additional displays from my parts box include a round
display, a 2.5" e-paper display, and some monochrome displays. 

Prototyping work is using SPI for the display and IIC for the client.

### Protocol coding

I have implemented a 'slave' IIC interface for the STM32 in C, and am looking
at what that might take for non-STM32 processors. See Hardware Progress below 
for some details around processor selection that were unexpected. 

I have started a device SPI interface for the STM32 as well. The open question
here is the response time. SPI sends octets as it receives octets and the
processor needs to wire up a DMA channel to ensure the bytes are avaiable on
time. It is unclear if I could implement a 'soft' client side SPI interface.

### Software bring up and debug

I have available in my equipment both test equipment that will decode these
buses as well as a [Glasgow](https://glasgow-embedded.org) which lets me
simulate either a client or a host for further debugging. 

## Hardware Progress:

### Processor selection(s)

I ran into an interesting challenge when looking for the ideal procesor. That
challenge is a low cost part that supports two SPI ports, two IIC ports, and
at least one asynchronous serial port. 

I've been planning to have ports for the display and I/O hardware and a second
set of ports that were available to the client to talk to the gizmo. Various
processors have these in theory but often only run them out to pins on their
higher pin count processors.

Another consideration is that on the most inexpensive implementation of SPI Term
I was planning a simple n x m keyboard scanning setup plus two quadrature
timer inputs for x \& y knobs. In various spec sheets the pins that are IIC and
SPI are also GPIOs if you're using them as GPIOs. So I can be creative in
multiplexing pin usage or using higher priced parts or using parts from vendors
like Nordic which have a more flexible I/O pin assignment scheme than ST does.

### Prototyping

I've been more succesful in prototyping because I have a number of development
boards and such that I've accumulated over the years. Keeping in mind parts that
are no longer discontinued, I have designed a prototype PCB which will allow me
to plug in any of the ST Micro 'Nucleo' dev boards and routes the I/O pins to
three connectors for a choice of display, as well as simple memory bus for the
FPGA implemented display.

Uncertainty in tariffs caused me to delay getting that proto board made however
I expect to have it in hand during July now that I have more confidence that I
won't blow my budget getting just the prototype board made.

## Next steps

The proto dev board is the top item on my next steps. I can continue to do
software and protocol development with my jumper wire setup. I am about a
month behind from my original internal schedule having found that it wasn't
simple to get all the ports I needed, and some of the design work taking
longer than expects. 

Items two and three are protocol implementations for both IIC and SPI, as
well as an 800 x 450 display (16:9) for HDMI using the ECP5 or Altera MAX10
FPGA. Having a 1280 x 720 display is the goal for the high end unit with a
stretch goal of a full 1920 x 1080. These are simplified somewhat because they
are DVI over HDMI and do not include HDCP or Audio.
