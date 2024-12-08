Dedicated I/O Device for Small Computers
========================================

This project is to create a "standard" display device which offloads the
common display things that someone writing programs on resource constrained
systems would otherwise have to do themselves.

This project originated from the observation that early microcomputers based
on the Z-80, 8080A, and 6800 were useful, in part, because you could use them
with a terminal. It wasn't until the Apple ][, IBM PC, Commodore-64, and
the Atari 5200 that the computer included a built in display generator. But
even there, the display generator did a lot of work that would have otherwise
fallen to the central CPU to do. As a result, an Arduino, with an Atmel 
ATMega 328P chip, would have enough resources to run BASIC but could not do
that an simultaneously run a display. Interaction through a 'Serial Monitor'
turned a $500+ desktop or laptop computer into a computer terminal. What
was more every display is a little bit different, whether it is the controller
that controls the display itself, or the I/O pins that are attached, or
the communication format which might be i2C, SPI, or parallel data. 

CP/M, Macintosh, and
Amiga systems were both powerful and fun to program but had resources
that are often less than what a typical SoC controller has. 
