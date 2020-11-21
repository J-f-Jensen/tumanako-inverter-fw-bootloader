--------------------------------------------
-- STM32 Bootloader
--
-- Johannes Hübner <dev@johanneshuebner.com>
--------------------------------------------

This boot loader does not interact with the built-in STM32 boot loader. It 
uses an interface of your choice, right now this is USART3. The 
interface flexibility and the independence from the BOOT pins are the 
main reasons for implementing this boot loader.

New update protocol
- 115200-8-N-2 (2 stop bits necessary when using a ZigBee module)

1 Send '2' indicating version 2 bootloader, wait about 500ms for magic byte 0xAA
2 If no reply goto 7
3 Send an 'S' indicating that it is awaiting an update size in pages
4 If no reply within about 500ms go to step 7
4.1 otherwise send a 'P' indicating that it is awaiting the actual page 
as a binary image
4.2 When page not received within about 1s, print 'T' and keep waiting
5 When page received send a 'C' indicating that it is awaiting the pages 
checksum
6 When checksum is correct and more pages need to be received, go to 
step 4.1
6.1 if all pages have been received go to step 5
6.2 When checksum isn't correct print an 'E' then go to step 4.1
7 When done print a 'D' and start main firmware

Notes:
- By checksum I mean the one calculated by the STMs integrated CRC32 unit.
- The actual firmware has a reset command the cycle through the bootloader
- The main firmware must be linked to start at address 0x08001000
- The bootloader starts at address 0x08000000 and can be 4k in size 
(right now its around 2.5k)

--------------------------------------------
-- STM32 Bootloader Updater
--
-- Johannes Hübner <dev@johanneshuebner.com>
--------------------------------------------

To be able to upgrade the boot loader code without a JTAG programmer an
upgrade application was developed. It follows the same protocol as the
boot loader but with different timing:

1. Keep printing "2" until magic and a page count <= 4 is received
2. Do the update process except the received pages are written to base address 0x08000000
3. Wait for "reset" command

To upgrade the bootloader, do the following, either using the web interface
or the python script
1. Send file stm32_bootupdater.bin
2. Send file stm32_loader.bin
3. Send file stm32_sine.bin

--------------------------------------------
-- Compiling
--------------------------------------------
You will need the arm-none-eabi toolchain: https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads
The only external depedency is libopencm3 which I forked. You can download and build this dependency by typing

make get-deps

Now you can compile stm32-loader and stm32-bootupdater by typing

make
cd bootupdater
make

And upload it to your board using a JTAG/SWD adapter, the updater.py script or the esp8266 web interface

Boot loader is on 0x08000000 and firmware at 0x08001000

First time upload on a fresh bluepill, connect USB-Serial RX/TX to PA_9 (TX) and PA_10 (RX), move boot jumper 0 and use stm32flash to upload the boot loader file, remeber to move the boot jumper back before resetting the board: 
stm32_loader.bin

Switch RX/TX to pin PB_10 (TX) and PB_11 (RX) and use update.py or ESP web interface to upload the firmware

Note for Windows users: Pyton serial interface is not installed as default in Windows, run the following command in a command prompt to install it:
pip install serial

