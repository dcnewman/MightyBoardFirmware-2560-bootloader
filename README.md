License
=======

This program is free software; you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation; either version 2 of the License, or any later version.

This program is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

About This Distribuion
======================

This is the standard ARDUINO STK500V2 bootloader, used on the Atmega 2560 and
other related processors and boards.  No changes were made to the actual 
source file of the bootloader.  Changes in this distribution are simply to
the make file.

The Mightyboard is an all-in-one 3D printer motherboard designed by Makerbot
and used in their Replicator 1, 2, and 2X montherboards.

In order to prevent unintentional resets, the Mightyboard uses a unique reset
mechanism:

[From the 8U2 Reset Function Testing document](https://github.com/makerbot/MightyBoardFirmware/blob/master/bootloader/8U2_firmware/8U2_Reset_Function_Testing.markdown):
 
>  We are using firmware for the 8U2 chip that was developed by the Arduino team.
> This firmware translates virtual Serial DTR line toggles to a RESET output that 
> connects to the reset pin on the Atmega1280. Serial implementations on Mac OSx 
> and Linux pull the DTR line down on connect and release it on disconnect.
>
> Mirroring the DTR to the 1280 reset means that the Bot will reset when 
> it is connected to ReplicatorG. This is undesireable behavior if the Bot 
> is in the middle of a print.
>
>The new firmware ignores virtual DTR line changes. It pulls the RESET 
>line down when it recieves a new connection with a baud rate of 56700. 
>57600 is the baud rate we use for firmware updates. The RESET line is 
>set high for all other baud rates. Normal communication with the bot 
>uses 115200 baud.

Normally, this stk500v2 bootloader runs at 115200.  In order to build a
version that runs at 57600 (as expected by ReplicatorG and the 8u2 firmware)
the Makefile was modified in the following manner:

1. A new target "mighty2560" was added to the Makefile, duplicating the mega2560 target.
2. The BAUDRATE and UART_BAUDRATE_DOUBLE_SPEED definitions were added to this target
3. The no-inline-small-functions option was added to insure that the compiled bootloader was less than 8k as required.

Instructions
============

1.  Clean up the source directory

	make clean

2.  Build the bootloader.  

	make mighty2560
	
	Note that the makefile is also modified to use the avr-gcc compiler located in the arduino distribution (/Applications/Arduino.app/Contents/Resources/Java/hardware/tools/avr/bin/avr-gcc) modify the Makefile if your avr-gcc toolchain is located somewhere else.  Note that this firmware has been built with avr-gcc 4.3.2, and has not been tested with other versions of the toolchain.

3.  Unlock the memory on your target board and erase the flash (if nessessary).  You can also set the fuse bits at the same time.  I used a USBASP programmer and a command similar to:

	avrdude -v -p m2560 -c usbasp -P usb -Ulock:w:0x3F:m -Uefuse:w:0xFD:m -Uhfuse:w:0xD8:m -Ulfuse:w:0xFF:m -e

3.  Load to your 2560 with the bootloader, and set proper lock bits: 

	avrdude -v -p m2560 -c usbasp -P usb -Uflash:w:stk500boot_v2_mega2560.hex:i -Ulock:w:0x0f:m
