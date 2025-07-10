# Scoreboard Server

This repository provides the code for an electronic cricket scoreboard. A Raspberry Pi hosts a small web interface that sends scores over serial to an Arduino, which drives multiple seven‑segment displays.

## Directory layout

- **software/scoreboard** – Arduino sketch `scoreboard.ino` controlling the display hardware.
- **software/CmdMessenger** – Serial messaging library used by the sketch.
- **software/ShifterStr** – Library for driving cascaded shift registers.
- **software/Streaming** – Provides C++ streaming operators.
- **software/arduinobase64** – Base64 implementation required by CmdMessenger.
- **software/web** – PHP pages, JavaScript and shell scripts making up the Raspberry Pi side.
- **software/raspberry.tgz** – Tarball containing `/usr/local/bin` utilities and the `/var/www` web root used on the Pi.

## Arduino sketch

`scoreboard.ino` listens for commands from the Pi using the CmdMessenger protocol. The header lists its history and dependencies:
```
//////////////////////////////////////////////////////////////
//
// Name: scoreBoard.pde
// Author: Ian Nice - ian@woscc.org.uk
// Version: 0.1 - 14/03/2013 - First version for testing
//          0.2 - 01/04/2013 - Updated to produce additional debug
//          0.3 - 06/04/2013 - Updated to simplify wiring and fix bug
//          0.4 - 07/04/2013 - Updated to use strings instead of integers
//          0.5 - 11/04/2013 - Changed to simplify wiring setup, now that i dont have limitation of Integer size.
//          0.6 - 02/07/2013 - Updated because ted doesnt like preceeding zeros :-)
//          0.7 - 20/02/2016 - Updated to work with Arduino IDE 1.6.7 - Thanks to James W @ Potton Town CC for helping debug the problems
//
// Acknowledgement:
//  shifter.h - http://www.proto-pic.com
//  CmdMessenger.h - https://github.com/dreamcat4/cmdmessenger
//  Streaming.h - http://arduiniana.org/libraries/streaming/
//  Base64.h - https://github.com/adamvr/arduino-base64
//  Where the idea began.......
//  http://www.fritz-hut.com/arduinopi-web-based-controller-for-arduino/
//
```

The sketch updates three groups of displays for batsmen scores, total, wickets, overs and target. It also supports a test mode activated by sending command `5#`.

## Web interface

The `software/web` directory contains the files served by the Pi. `index.htm` provides a mobile‑friendly page that uses jQuery Mobile and Mobiscroll widgets to enter scores. Every change posts to `scoreboard.php`, which forwards the data to the Arduino via the serial port.

An excerpt of `scoreboard.php` shows how the command string is built:
```php
<?php
	include "php_serial.class.php";

	$totalHundreds=$_POST['totalHundreds'];	
	$totalTens=$_POST['totalTens'];	
	$totalOnes=$_POST['totalOnes'];	
	$wicketsOnes=$_POST['wicketsOnes'];	
	$oversTens=$_POST['oversTens'];	
	$oversOnes=$_POST['oversOnes'];	
	$batsmanaHundreds=$_POST['batsmanaHundreds'];	
	$batsmanaTens=$_POST['batsmanaTens'];	
	$batsmanaOnes=$_POST['batsmanaOnes'];	
	$batsmanbHundreds=$_POST['batsmanbHundreds'];	
	$batsmanbTens=$_POST['batsmanbTens'];	
	$batsmanbOnes=$_POST['batsmanbOnes'];	
	$targetHundreds=$_POST['targetHundreds'];	
	$targetTens=$_POST['targetTens'];	
	$targetOnes=$_POST['targetOnes'];	

	$total=$totalHundreds.$totalTens.$totalOnes;
	$overs=$oversTens.$oversOnes;
	$batsmana=$batsmanaHundreds.$batsmanaTens.$batsmanaOnes;
	$batsmanb=$batsmanbHundreds.$batsmanbTens.$batsmanbOnes;
	$target=$targetHundreds.$targetTens.$targetOnes;


	$serial = new phpSerial;
	$serial->deviceSet("/dev/ttyACM0");
	#$serial->deviceSet("/dev/ttyAMA0");
	#$serial->confBaudRate(115200);
	$serial->confParity("none");
	$serial->confCharacterLength(8);
	$serial->confStopBits(1);
	$tempString="4,".$batsmana.",".$total.",".$batsmanb.",".$wicketsOnes.",".$overs.",".$target."#";
	#echo($tempString."<br>");
	$serial->deviceOpen();
	$serial->sendMessage($tempString);

```

Shell scripts located in `usr/local/bin/scoreboard` assist with managing the serial device and system power. For example, `checkShutdown.sh` triggers a system shutdown when a flag file is present:
```bash
#!/bin/bash
if [ -f /var/www/shutdown/shutdown.server ]; then
  rm -f /var/www/shutdown/shutdown.server
  /sbin/shutdown now 
fi
```

The `raspberry.tgz` archive packages these scripts and the website for easy deployment on a Pi.

## Getting started

1. Upload `software/scoreboard/scoreboard.ino` to the Arduino.
2. Extract `software/raspberry.tgz` on the Raspberry Pi so the web files reside in `/var/www` and the helper scripts in `/usr/local/bin/scoreboard`.
3. Connect the Arduino via USB (`/dev/ttyACM0` by default).
4. Open the Pi's web page from a browser and update the scores.

## License

Third‑party libraries retain their original licenses. No specific license is provided for the scoreboard code.
