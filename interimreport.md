SPI Terminal Interim Report
===========================

## Design Progress:

The design process is focused on defining the requirements for the underlying
support hardware, and the register definitions for the resulting device. This
is done in the form of "threshold" type requirements where the minimums are
defined and a path for creating more capable devices is charted out so that
people building new SPI Terms from a new design can be compatible with the
existing ecosystem.

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

## Software Progress:

## Hardware Progress:
