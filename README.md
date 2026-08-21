bb_epaper (BitBank epaper library)<br>
----------------------------------
Project started 9/11/2024<br>
Copyright (c) 2024 BitBank Software, Inc.<br>
Written by Larry Bank<br>
bitbank@pobox.com<br>
<br>
<b>What is it?</b><br>
A frustration-free e-paper library suitable for Arduino, Linux, or random embedded systems with no OS.<br>
<br>
<b>Why did you write it?</b><br>
I've always had an interest in graphics and displays. After experimenting with epaper, I looked around for code to control them. All I found was half-implemented demos and frustration. I wrote my own support for epaper panels into my OneBitDisplay library, but it made it a bit unwieldy. E-paper is in a class by itself, so I decided to create a unique library for working with these panels. The main goal was to reduce frustration and make an efficient set of functions that work with a large collection of panels in a consistent way.<br>
<br>
<b>What's special about it?</b><br>
It's the first epaper library I've ever seen which can draw text and graphics without needing a local copy of the image data. There are some limitations to working that way, but it opens the possibility to control large displays on MCUs with nearly no memory. It also supports keeping a local copy of the graphics in RAM (the normal way to work with epaper). By offering both options, this library is unique.<br>
<br>
<b>A word about the displays</b><br>
One of the most challenging aspects of using epaper displays is that there are many displays which look the same, but use different internal controllers. Even ones that use thhe same company's chip can be using different variants with ever changing instruction sets. The display controller can also be permanently damaged by sending it the wrong commands or the wrong settings. I created a chart to try to make sense of the various panels, but please use caution when selecting the right bb_epaper display type:
<br>
Features:<br>
---------<br>
- C API and C++ wrapper class with all of the usual graphics functions<br>
- Supports any bit depth of graphics (currently 2/3/7/16 color & grayscale)<br>
- Supports a large number of panels in a consistent way, without tons of repeated code<br>
- Supports any number of simultaneous displays of any type (mix and match)<br>
- Includes support for a unique compressed font and bitmap format<br>
- Includes command line tools to convert and compress TTF fonts and WinBMP files<br>
- Text cursor position with optional line wrap<br>
- A function to load a Windows BMP file<br>
- Optimized pixel pipeline customized for each supported bit depth<br>
<br>
The compressed font handling depends on my Group5 data compression library (included).<br>
See the Wiki for help getting started<br>
https://github.com/bitbank2/bb_epaper/wiki <br>
<br>

A few words about fonts<br>
-----------------------<br>

The library includes 4 fixed fonts (6x8, 8x8, 16x16 and 12x16). The 16x16 and 12x16 are really a stretched+smoothed version of the 6x8 to save FLASH space. To use more elaborate fonts with more extensive character support, use my BB_FONT format compressed bitmap fonts. This functionality is part of my Group5 compression library.<br>

Black & White (& Red)<br>
------------------------<br>
The current code supports 1-bit black and white epaper displays as well as the B/W/R models. At the time of this writing, the BWR models can only do a full refresh (no fast nor partial). This may change in a future version.<br>
<br>
See WiKi and example code for how to use the library.<br> 
<br>

A unique feature of the library is that you can set the panel type at run-time. I created a testing tool which makes use of this feature (photo below). It allows you to run diagnostic tests on any of the supported panels and hot-swap them without restarting.
<br>
![bb_epaper](/ep29rv2_hardware.jpg?raw=true "WeAct 2.9\" 128x296 module + ESP32-C3 SuperMini")
<br>
Hardware: WeAct Studio 2.9" 128x296 B/W/R e-paper module (GDEM029C90, SSD1680) driven by an ESP32-C3 SuperMini. Pins: CS=7, DC=9, RES=8, BUSY=5, MOSI=6, SCLK=4.
<br>
![bb_epaper](/ep29rv2_bwr_test.jpg?raw=true "B/W/R test pattern")
<br>
![bb_epaper](/ep29rv2_bwr_lines.jpg?raw=true "B/W text lines with red numbers")
<br>
Full-screen black/white/red fills run correct with no Y offset or garbage rows; alternating black/red text lines display correctly across the whole panel.
<br>

New in this fork: EP29Rv2_128x296
----------------------------
A new panel type `EP29Rv2_128x296` has been added for the 2.9" 128x296 Black/White/Red e-paper display (GoodDisplay **GDEM029C90**, SSD1680 controller).

This is a purely additive change - the existing `EP29R_128x296` and `EP26R_152x296` entries are left untouched.

<b>What was added</b>
- New enum value `EP29Rv2_128x296` in `src/bb_epaper.h`, just before `EP_PANEL_COUNT`
- New init sequence `epd29rv2_init_sequence_full` in `src/bb_ep.inl`, matching `GxEPD2_290_C90c::_InitDisplay()`:
  - `0x21` (Display Update Control 1) sent with **2 data bytes** (`0x00, 0x80`) - the SSD1680 expects 2 bytes here; sending only 1 desynchronises the rest of the init sequence
  - `0x3C` border waveform = `0x05`
  - RAM X end = `0x0F` (128 px), RAM Y end = `0x0127` (296 rows)
  - RAM Y counter starts at 0
- New `panelDefs[]` entry: `{128, 296, 0, epd29rv2_init_sequence_full, NULL, NULL, BBEP_3COLOR, BBEP_CHIP_SSD16xx, u8Colors_3clr}`
- `bbepRefresh()` special case for `EP29Rv2_128x296` using display update control `0xF7` (full LUT refresh, same as GxEPD2_290_C90c `_Update_Full()`), instead of the default 3-color `0xC7`

<b>Why not reuse EP29R_128x296?</b>
`EP29R_128x296` exists but its init sequence has issues on this panel:
- `0x21` sent with only 1 data byte (SSD1680 requires 2)
- master activation (`0x20`) commented out
- RAM Y counter initialised to the last row instead of 0

Those are left as-is to avoid regressions for anyone currently using `EP29R_128x296`; the new panel type provides a correct, tested path for the GDEM029C90.

<b>Usage</b>
```
BBEPAPER bbep(EP29Rv2_128x296);
```

<b>Tested</b>
- Hardware: WeAct Studio ESP32-C3 + 2.9" 128x296 B/W/R panel (GDEM029C90)
- Pins: CS=7, DC=9, RES=8, BUSY=5, MOSI=6, SCLK=4
- Full-screen black/white/red fills: correct, no Y offset, no garbage rows
- Alternating black/red text lines across the whole display: correct
- Text + circles + bitmap (three_color example content, rotation 270): correct
<br>

Thank you to the Open Home Foundation for being a sponsor of this work:<br>
https://www.openhomefoundation.org/<br>

If you find this code useful, please consider sending a donation or becoming a Github sponsor.

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=SR4F44J2UR8S4)

