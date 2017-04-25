# About Phoenix Thermal Printer
The Phoenix Thermal Printer by Pyramid Technologies provides a ESC/POS compatible printer interface over DB9 or USB connections. True Windows printer models are also available for XP through Windows 10.

## What you need
1. Phoenix Printer
2. Proper harness for your model (see Firmware section below)
3. A serial emulator program

## Firmware and Harness
**IMPORTANT:** You must use the appropriate firmware and cable in order for this to function correctly. Look on the bottom of your Phoenix Printer to determine the model of your device. Refer to the table below to verify what you need:

| Model | Firmware | Cable | Driver Required | Notes | 
|-------|----------|-------|-----------------|-------|
| Pulse | Any* | No | DB9 | ESC/POS over DB9 only. USB is only used for configuration |
| Pulse | S08 (Full ESC/POS) | DB9 | No | ESC/POS over DB9 |
| E-PHX | E-PHX (Full ESC/POS over USB) | USB A to B | [Yes](https://pyramidacceptors.com/apps/PTI_Unified_Serial_Driver_Installer.zip) | ESC/POS over USB|
| USB | Win USB | USB A to B | [Yes](https://pyramidacceptors.com/apps/PTI_Phoenix_Windows_Driver_109.zip) | Does not use serial port, true Printer USB class

**IMPORTANT:** If you are using DB9, we recommend [our harness](http://shop.pyramidacceptors.com/phoenix-rj-45-to-db-9-harness-phx-5p002/).

## Setup
At this point, we assume that you have your Phoenix Printer with the appropriate harness. If not, contact our support team and we'll get you on track.

Launch your favorite serial emulator and configure your port for 9600 baud, 1 start bit, 1 stop bit, no parity. No handshaking settings are required or supported on the Phoenix.  

_Note_: For DB9 connection, you may manually override the serial settings via Phoenix Tools under the Comm Settings -> Baud & Parity menu. The default 9600 is great for testing.  


## ESC/POS Basics
ESC/POS is a protocol for sending data to printers. The basic commands are more or less standardized across vendors but there are slight variations in the more complex command sequences. We will be covering the simple, generic commands such as font styling. In our examples, we will be transmitting ESC/POS data as hexidecimal through our serial emulator because it is more consistent than jumping between ASCII, decimal, and hex.

From the host point of view (yours), there is no flow control or CRC commands to worry about. Simply transmit you commands and let the Phoenix handle the rest. If there are errors in your commands, ESC/POS is the type of protocol that silently fails. That means that malformed commands will be ignored. As a rule of thumb, start with single commands and then chain together more and more.

ESC/POS commands are processed in the order in which they are received. A command can _only_ affect data that is sent after the command. In the following examples, we will use plain english instead of the ESC/POS codes. The following line demonstrates the order of operations in ESC/POS.  

`[bold command] [some text] [italic command] [print command]`

This only prints "some text" in bold. No text data was sent after the italic command so no text was in the buffer to be formatted.  Now, we continue sending commands... 

`[some more text] [print command]`  

The output will be both bold and italic. ESC/POS will keep applying formats until you disable the modification or reset the printer. For example...

`[disable bold] [some other text] [print command]`  

This will print in only italic.

`[initialize printer] [some text] [print command]`  

This will print in the default format.

One last item to note is that you must call the print buffer and feed line command. As the name implies, this will output all received, formatted data to the print head. If the line's width exceeds that of the print head, the Phoenix will automatically line wrap the text for you.

The Phoenix by default will auto feed and cut tickets once these 2 conditions are met:  
1. 3 printed lines  
2. Nothing has been received by the print buffer in 30 seconds  

_Note_: This auto feed and cut feature can be turned off for testing under "Comm Settings" -> "ESC/POS Options"  -> "Auto Cut Enabled" 

## Sending ESC/POS
Congratulations! You're ready to start printing. Try out these simple commands to get feel for how ESC/POS works.

Print "Hello, Phoenix printer!"  
`48 65 6c 6c 6f 2c 20 50 68 6f 65 6e 69 78 20 70 72 69 6e 74 65 72 21`

Print the buffer, feed 1 line  
`0A`

Enable underline  
`1B 2D 01`

Print "This is underlined" (sorry no markdown for underline)  
`54 68 69 73 20 69 73 20 75 6e 64 65 72 6c 69 6e 65 64`

Print the buffer, feed 1 line  
`0A`

Enable double height  
`1D 21 01`

Disable underline  
`1B 2D 00` 

Print ##"This is double height"  
`54 68 69 73 20 69 73 20 64 6f 75 62 6c 65 20 68 65 69 67 68 74`

Print the buffer, feed 1 line  
`0A`

Reset to normal character height  
`1D 21 00` 

Enable right justify  
`1B 61 02`

Print "Right justified"  
`52 69 67 68 74 20 6a 75 73 74 69 66 69 65 64`

Print the buffer, feed 1 line  
`0A`

## Going Further
ESC/POS is a huge protocol with many varities and we work hard to keep our implementation consistent. A complete PDF outlining our command set is available here: https://pyramidacceptors.com/pdf/PHX_ESC_POS_Protocol.pdf

If you have question please feel free to create an issue/question on Github or contact our support team.
