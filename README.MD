# A collection of Dreamcast game patches to enable serial PPP

**As of 2026-09-16 patched games now use hardware flow control (RTS/CTS). Please ensure your adapter includes those extra signal lines**

The Sega Dreamcast has a rear serial port that can be used for various applications. There is official, but largely undocumented support for using the serial port for an external modem. Dreamkey 3.0 and later Dream Passport browsers can be configured to use the serial port to connect online. 

Since Dreampi 2.0, the Dreampi script will check for a USB serial port and monitor it for Hayes AT commands related to modem initialization and dialing. If it detects a dial command, it will provide the appropriate responses to the Dreamcast and attempt to start a PPP session.

Two games have out-of-the box serial PPP functionality - Alien Front Online and Propeller Arena.
Many in-game browsers also have serial PPP functionality.

The patches on this page generally replace or redirect modem related code and force the bytes to be transported over the SCIF (serial) interface. The complexity of these changes varies from game to game. Some games have a fully functional code path for the serial port, so the patches are only a few changed bytes while others need more supporting code. In those cases where more support code was needed, the KOS SCIF implementation was used as the reference.

Disclaimer: ChatGPT 5.6 was employed in developing these patches.

## Patch inventory

“Bytes affected” is the byte-for-byte difference between the source file and the patched file(s), summed across every file patched; it includes added or removed bytes when a file changes size. For rebuilt WinCE binaries, this count includes incidental binary-layout changes from rebuilding the binary and therefore is larger than the amount of new networking logic. The Rune Jade package also includes the English translation and server patch, so most of its affected bytes are unrelated to serial support.

| Game | File(s) patched | Bytes affected | High-level summary |
|:---|:---|---:|:---|
| 4x4 Evolution (NTSC-U) | `0WINCEOS.BIN` | 3,061 | Configures and activates the WinCE COM1 serial driver at 115200 baud so the stock RAS/modem stack communicates through SCIF. |
| ChuChu Rocket! (NTSC-U) | `1ST_READ.BIN` | 3,778 | Replaces the physical modem DTE transport with a 115200-baud SCIF driver while retaining the game's existing Hayes and PPP layers. |
| Gundam: Federation vs. Zeon (NTSC-J, Disc 1) | `1ST_READ.BIN` | 18 | Redirects the game's modem transport to its existing SCIF implementation and selects the 115200-baud serial configuration. |
| Gundam: Federation vs. Zeon DX (NTSC-J, Disc 2) | `1ST_READ.BIN` | 18 | Redirects the game's modem transport to its existing SCIF implementation and selects the 115200-baud serial configuration. |
| The Next Tetris: On-line Edition (NTSC-U) | `0WINCEOS.BIN` | 266,299 | Rebuilds the WinCE image with COM1 registered and activated at 115200 baud, allowing the stock dial-up stack to use SCIF. |
| Ooga Booga (NTSC-U) | `1ST_READ.BIN` | 1,602 | Replaces the physical modem DTE layer with a 115200-baud SCIF transport while preserving the stock modem and PPP behavior. |
| Outtrigger (NTSC-U) | `1ST_READ.BIN` | 8 | Selects the game's existing serial communications path and corrects its send-readiness behavior so gameplay traffic leaves through SCIF. |
| Power Smash (NTSC-J) | `1ST_READ.BIN` | 1,775 | Substitutes a 115200-baud SCIF DTE using the game's original DTE storage and adds the missing LCP echo-response handling. |
| Pro Yakyuu Team de Asobou Net! (NTSC-J) | `NET_DLL.BIN` | 1,621 | Replaces the network library's physical modem DTE with a 115200-baud SCIF implementation while retaining its Iwango protocol stack. |
| Quake III Arena (NTSC-U) | `QUAKE3/Q3_USA.BIN` | 3,591 | Replaces the modem's low-level DTE API with a KOS-derived 115200-baud SCIF driver while leaving Quake III's modem, Hayes, and PPP layers intact. |
| Rune Jade (NTSC-J, English + server patch) | `0WINCEOS.BIN`, `ADC/MAIN.ADO`, `OTHER/E_CM.PVR`, `OTHER/G512.PVR`, `OTHER/G512_2.PVR`, `OTHER/NET1.PVR`, `OTHER/NML.PVR`, `OTHER/SE_CHR1.PVR`, `OTHER/TITLE.PVR`, `WINCE/IWLOBBYENGINE.DLL`, `WINCE/RUNE.EXE` | 1,820,262 | Combines the English and replacement-server changes with a WinCE COM1/115200 configuration that routes dial-up networking through SCIF. |
| Sega Rally 2 (NTSC-J) | `0WINCEOS.BIN`, `DWANGO.DLL` | 5,749 | Registers COM1 at 115200 baud with hardware handshaking and redirects the DWANGO dial-up device selection from the modem to serial. |
| StarLancer (NTSC-U) | `0WINCEOS.BIN` | 336,937 | Rebuilds the WinCE image to activate COM1 at 115200 baud and routes the stock Microsoft PPP/modem stack through the serial driver. |
| Unreal Tournament (NTSC-U) | `1ST_READ.BIN`, `UTDC.BIN` | 7 | Enables RGB-compatible startup and redirects Unreal Tournament's modem-facing transport selection to its existing SCIF path. |
| Cyber Troopers: Virtual-On Oratorio Tangram (NTSC-J) | `1ST_READ.BIN` | 1,774 | Replaces the physical modem DTE with a 115200-baud SCIF implementation and supplies LCP echo responses for reliable link cleanup. |
| Worms World Party (NTSC-U) | `0WINCEOS.BIN` | 175,204 | Rebuilds the WinCE image with the serial driver activated and COM1 selected at 115200 baud for the stock dial-up stack. |

## Requirements
* Original game GDI
* USB-serial adapter AKA Coder's Cable (see details below)
* Universal Dreamcast Patcher: https://github.com/DerekPascarella/UniversalDreamcastPatcher
* Dreampi or Dreampi VM running version 2.0+
	* Highly recommended to opt into auto updates for the support scripts so serial PPP updates get received.
	* You can do this by deleting noautoupdates.txt on the rood of the SD card or at /boot directory.
 
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
	 
## Connection Guide
|Dreamcast|Micro SD Sniffer|Serial Adapter|
|:---|:---|:---|
|TX|CMD|RX|
|RX|DAT0|TX|
|GND|GND|GND|
|CTS|CLK|RTS|
|RTS|CD|CTS|


If you want to solder directly to a serial port connector, refer to this pinout:

![Screenshot](https://github.com/eaudunord/dc-taisen-netplay/blob/main/DC_SD_Adapter_V2.jpg?raw=true)

## Connecting online
1. Make sure your adapter is plugged into the Dreampi or attached to the VM before booting up or running the Dreampi service
2. Follow the in-game menus to connect online. The game will treat a serial connection as a modem connection.

## Note about officially compatible games
Games with official serial port support need to be configured to use it with a web browser. The best way to do this is with XDP or Dreamkey 3.0. The default settings of 115200 baud 8-N-1 are correct.

The patch for Propeller arena changes its baud rate setting to 115200 so it becomes compatible with the Dreampi implementation.

## Testing Status

|Game|Region|Server connection|Multiplayer|Notes|
|:---|:---|:---|:---|:---|
|4x4 Evolution|NTSC-U|Yes|||
|ChuChu Rocket|NTSC-U|Yes|Yes|Input delay still exists|
|Gundam Federation vs Zeon|NTSC-J|Yes|||
|Next Tetris|NTSC-U|Yes|||
|Ooga Booga|NTSC-U|Yes|Yes|4 serial players extremely smooth|
|Outtrigger|NTSC-U|Yes|||
|Power Smash|NTSC-J|Yes|||
|Pro Yakyuu Net|NTSC-J|Yes|||
|Quake 3|NTSC-U|Yes|Yes|3 player tested|
|Rune Jade|NTSC-J|Yes||Apply to clean NTSC-J rip|
|Sega Rally 2|NTSC-J|Yes|||
|Starlancer|NTSC-U|Yes|||
|Unreal Tournament|NTSC-U|Yes|||
|Virtual On|NTSC-J|Yes|||
|Worms World Party|NTSC-U|Yes|||
