# A collection of Dreamcast game patches to enable serial PPP

The Sega Dreamcast has a rear serial port that can be used for various applications. There is official, but largely undocumented support for using the serial port for an external modem. Dreamkey 3.0 and later Dream Passport browsers can be configured to use the serial port to connect online. 

Since Dreampi 2.0, the Dreampi script will check for a USB serial port and monitor it for Hayes AT commands related to modem initialization and dialing. If it detects a dial command, it will provide the appropriate responses to the Dreamcast and attempt to start a PPP session.

Two games have out-of-the box serial PPP functionality - Alien Front Online and Propeller Arena.
Many in-game browsers also have serial PPP functionality.

The patches on this page generally replace or redirect modem related code and force the bytes to be transported over the SCIF (serial) interface. The complexity of these changes varies from game to game. Some games have a fully functional code path for the serial port, so the patches are only a few changed bytes while others need more supporting code. In those cases where more support code was needed, the KOS SCIF implementation was used as the reference.

Disclaimer: ChatGPT 5.6 was employed in developing these patches.

## Requirements
* Original game GDI
* USB-serial adapter AKA Coder's Cable (see details below)
* Universal Dreamcast Patcher: https://github.com/DerekPascarella/UniversalDreamcastPatcher
* Dreampi or Dreampi VM running version 2.0+
 
## About the adapter
The easiest way to get up and running hardware-wise would be to acquire the following:
 * Dreamcast SD Adapter
 * SD card sniffer (micro or full size)
 * USB to TTL serial adapter with a *cp2102n* chipset. This is the most reliable chip I've tested so far
![Screenshot](https://github.com/eaudunord/dc-taisen-netplay/blob/main/Materials.jpg?raw=true)
 
Assembly:
 * Attach header row pins to the SD sniffer. Solderless press fit, or friction fit options are available if you can't solder.
 * Connect the appropriate wires from the USB-serial adapter to the corresponding pins using the chart below as a guide
 * Insert the SD card sniffer fully into the SD card slot on the SD adapter
 * Insert the SD adapter into the Serial port on the Dreamcast console
 * Note: A number of people have reported they need to ground CTS (CLK on the sniffer) to get things to work. 
     * If you do this, RTS from the USB adapter isn't needed.      
![Screenshot](https://github.com/eaudunord/dc-taisen-netplay/blob/main/ConnectionGuide.PNG?raw=true)


If you want to solder directly to a serial port connector, refer to this pinout:
![Screenshot](https://github.com/eaudunord/dc-taisen-netplay/blob/main/DC_SD_Adapter_V2.jpg?raw=true)

## Connecting online
1. Make sure your adapter is plugged into the Dreampi or attached to the VM before booting up or running the Dreampi service
2. Follow the in-game menus to connect online. The game will treat a serial connection as a modem connection.

## Note about officially compatible games
Games with official serial port support need to be configured to use it with a web browser. The best way to do this is with XDP or Dreamkey 3.0. The default settings of 115200 baud 8-N-1 are correct.

The patch for Propeller arena changes its baud rate setting to 115200 so it becomes compatible with the Dreampi implementation.