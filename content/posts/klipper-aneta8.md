+++
title = 'Installing Klipper on the Anet A8'
date = 2023-12-04T19:27:54+01:00
categories = ['3D printing']
+++

The Anet A8 is a bit of a relic of times gone by, what with the Anet having failed and generally being way behind modern printers in terms of features.

That said, I got mine in 2020 as my first printer, and I've since tuned it into a fine powerhorse. It's still going great, and I'm always looking for further ways to improve it.

## Why did I pick Klipper as the next upgrade?

The Anet has a 8-bit board, the ATmega1284P. I had long since upgraded Anet's stock (and faulty!) firmware into a more sanely configured Marlin 1.9, but the board itself prevented it from being a satisfying solution forever.

The board has a pretty limited memory: Marlin 2.0, the new version, couldn't fit on the board (well, [it could](https://www.crosslink.io/2020/08/14/shrinking-marlin-2-0-how-to-reduce-firmware-size-for-8-bit-boards-and-still-use-a-bltouch-and-the-filament-sensor/), but not while including all the features I needed or wanted to try out). I thinkered all I could with it, but this alread had me stuck on the old 1.9 version of the firmware, limiting improvements.

Upgrading the board itself is a hassle and a half, because it means changing and rewiring the whole printer, and I'd be better off buying a new printer at that point. 

I was just going to deal with tuning the old firmware as much as I could, but one of my recent upgrades to my homelab freed up a Raspberry PI4, which opened up a new avenue to try: Klipper.

[Klipper](https://www.klipper3d.org/) works around the board limitations (aside from memory, a 8-bit board is inherently slower and less powerful than most modern 32-bit boards, and therefore less precise) by only loading a very lightweight firmware on the printer. The Klipper firmware only takes care of moving mechanical parts and executing Gcode commands: all the calculations are done by a separate computer, connected to the board via USB/Serial and running the actual Klipper software.

It's also got several other features, but those alone are a really big upgrade to the Anet already. It also offers a web interface, makes the printer controllable via app, makes a lot of calibrations as easy as typing on your pc.

## Off to the races

I delegated the installation of Klipper on the Raspberry to KIAUH[^attribution], the [Klipper Installation And Update Helper](https://github.com/dw-0/kiauh). It's a neat little script that sets up for you klipper and its several plugins, and also takes care of helping you keep it updated in the future.

The Github page explains how to set up the Raspberry in detail. Once I had the script going, I went into the install menu and selected:

- **Klipper** and **Moonraker**, which are the strictly necessary components.
- **Fluidd** which is the interface I liked the most. You can pick between Fluidd, Mainsail and Octoprint (which is not made specifically for Klipper, but it's also the one the Klipper Docs usually refer to in their examples. 
    - I also wanted to use KlipperScreen to have a touch interface, but the touchscreen I had on hand decided it didn't like the new versions of PI OS anymore. It's a fight for another day.

- **Mobileraker's Compainon**, which optionally enables me to use the Mobileraker app to control the printer. I don't expect to use it very often, but I liked having the option.

That's it for the software side. The web interface is already up and can be found by typing the PI's IP address into any browser.

## Flashing the firmware 

The Anet had a peculiarity: some boards came shipping a bootloader, and some didn't. If you've ever upgraded it, you'll know which kind you are. If you don't, you'll find out when the command to upload the firmware doesn't work.

If you don't have the bootloader you need either an [USBasp](https://www.fischl.de/usbasp/), an Arduino board or another ISP programmer of your choice and the Arduno IDE. Even if you already have the bootloader as I did, I do suggest getting the USBasp. It's cheap and it gives you the reassurance that you can fix almost any mess you can make on the board.

### Building the firmware

Go back to the KIAUH menu and pick the "Advanced" option. From there, select "Build only", and dial in the two options from the new screen that pops up by selecting the AVR family of boards and the atmega1284p as the specific board. Then quit, save and let it compile.

The Anet can't be flashed with the standard method for Klipper, so you'll need to use the command provided in the [Anet A8 config file](https://github.com/Klipper3d/klipper/blob/master/config/printer-anet-a8-2017.cfg): `avrdude -p atmega1284p -c arduino -b 57600 -P /dev/ttyUSB0 -U out/klipper.elf.hex`

The board will need to be connected to the Raspberry via USB. Before issuing that command, hit the last option in the Advanced menu (get MCU info) and note down the long ID starting with `dev` it'll return, and replace `/dev/ttyUSB0` with it. Also save it for later, since you'll need it again.

### Flashing the bootloader

If the command can't contact the board, you're probably missing the bootloader. Grab your ISP programmer of choice and turn on the Arduino IDE. 

You'll need to go to `File > Preferences` and add `https://github.com/benlye/anet-board/raw/master/package_anet_board_index.json ` in the Additional Boards Manager field, before heading over to `Tools > Boards Manager` and installing both the `Arduino` and `Anet` libraries. 

Select the Anet board, the name of your programmer and click the option `Burn bootloader` from the tools menu.

## Last touches

Head over to the configuration section of the Web UI and open up `printer.cfg`. You'll see some lines already there; copy-paste under it the [Anet A8 config file](https://github.com/Klipper3d/klipper/blob/master/config/printer-anet-a8-2017.cfg) from earlier. 

There is one duplicate line between the two of them, the one under the `[mcu]` header. Delete one and fill the other in with the `/dev/` ID you got earlier from KIAUH. That's it!

Turn on the printer (keep it connected to the Raspberry). Don't worry if it doesn't show anything but squares - it won't until it starts talking to the main software. Click restart firmware in the main Fluidd page - and hopefully everything will start working.

The Klipper docs have a page listing all the [checks you should do](https://www.klipper3d.org/Config_checks.html) before printing to ensure everything is running smoothly. Run through it, and then start printing! 

[^attribution]: Special thanks to [POuL's 3D printing course](https://slides.poul.org/2023/stampa-3d/4-klipper/) for having set me on this project and having saved me lots of time with KIAUH!
