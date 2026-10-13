# Flashing a unit

This page takes a unit running the stock firmware and puts the Armbian
image from [BUILD.md](BUILD.md) on it.

## The three ways, by circumstance

***The Medium Way (stock image):***
see below

***The Hard Way (stock image + you fucked up):***
see section 2.1

***The Easy Way (post-flash):***
plug that motherfucker into USB like a raspberry pi pico while holding the reset button down. it will mount itself as a raw USB disk volume to your computer, then you can use BalenaEtcher or whatever your flasher of choice is to just spray that shit on. or: see section 3

## Before you start

- Flashing erases everything on the box's internal storage (eMMC).
- It also erases the vendor's serial number. If you want to keep it,
  back it up first; the [README](README.md) has the commands.
- Remove any installed storage drives, or accept the typo that stands between you and data loss.
- ***UNDERSTAND:*** This box is cheap for a reason. Its 5V power rails are not particularly well isolated nor are they particularly well controlled. If you plug a USB cable from a USB host device (like a PC), ***it will instantly power on, even with the 12V power cable disconnected.*** You must know this up front or you will be endlessly frustrated.
- This NAS has a debug UART port inside of the box. If you are curious what it's actually doing as you follow this process, feel free to crack it open and hook up an FTDI UART adapter to it. Just a warning: I literally tried 14 different serial adapter cables until I found one that worked. The $3 cheapos on Amazon can't even fathom 1500000 baud. Spend the $18 if you're curious, get something real.

## What you need

1. A YXK IronCow Zero1 NAS box.
2. A high quality USB A-to-A cable, preferably USB 3.0.
3. A Windows PC (you can do it on a Linux PC, but I haven't tried it. rkdeveloptool exists) with RKDevTool†† and Rockchip drivers††.
4. A paperclip, thumbtack, bobby pin, probably a toothpick but I didn't try, or a freakishly small twig of some kind.
5. The release image from the releases page, or a [custom-built image](BUILD.md).
6. (if you fuck it up) the rk3568-ddr4-emmc-evb-miniloader.bin recovery program image from the releases page.†
7. (if you fuck it up) a #2 Phillips head screwdriver plus a small spudger or a small flathead, like an eyeglass flathead or something. Or the stick, maybe. If it's powerful enough.
8. Prolly like an hour. Less if you're competitive about it, I bet.

Note: Files mentioned here will be available in the releases page.

## 1. Put the stock unit into loader mode

You have been personally kissed by the BSP engineers at Rockchip with the software on this device. You do not need any internal access to the hardware inside to completely erase and replace the stock OS image. The vendor kernel tree the image was compiled with supports RockUSB programming commands out of the box. It will happily provide you the opportunity to overwrite the entire onboard eMMC flash without issue, following the steps below:

***WITH THE 12V POWER ADAPTER UNPLUGGED, AND REMAINING UNPLUGGED, THE ENTIRE TIME***
1. Connect your USB A-to-A cable to a computer with the Rockchip Developer Tool software and Rockchip drivers preinstalled. DO NOT PLUG THE CABLE INTO THE NAS YET.
2. Launch RKDT.
3. With a paperclip, thumbtack, freakish stick you found in the woods, or whatever else you can find to fit into the recessed RESET button hole on the back of the unit situated between the HDMI port and the USB type-C port, press firmly on the RESET button inside of the housing. Do not apply excessive force, the button is not particularly durable, but you should feel it click firmly.
4. With your other hand, or with a third hand if one is available, insert the free end of the USB A-to-A cable already connected to your computer into the USB type-A port on the back of the unit (NOT the type-C port).
5. RKDT should say "Found One LOADER Device" on the Download Image screen.

Proceed to the next section.

## 2. Write the image

THIS SECTION IS IRREVERSIBLE. There is no significant chance for it to irrevocably brick the device, however, if you interrupt the flashing process, there is a VERY HIGH CHANCE that you will have to physically disassemble the back plastic housing of the unit to access the factory maskROM initial programming button (it's labelled "U-BOOT" and beneath the fan). This is a pain in the ass, and will cause your total process time to exceed an hour, most assuredly. ***Make sure the cable is fully connected at both ends, and both the computer and the NAS box are in a physically secure location***. Flashing will take about 3 minutes if you have a high quality USB 3.0 cable. It could take up to 16 minutes if you have a USB 2.0 cable, by my testing.

***WITH THE 12V POWER ADAPTER UNPLUGGED, AND REMAINING UNPLUGGED, THE ENTIRE TIME***
1. Make sure RKDT is open on the "Download Image" screen, and says "Found One LOADER Device" in the textbox at the bottom.
2. In the memory loader table, you should have exactly 1 row. Set the Path cell value to the path to your image binary file. To prevent issues with WIN32 path limits, I recommend you change the name of your image to something short, and place it right next to the RKDevTool.exe binary, so you can just set this cell to the short name of the file.
3. Make sure the Address cell has the value 0x00000000, and the program checkbox is ticked.
4. Click the Run button.
5. Wait between 3 and 16 minutes for the flashing process to complete.
6. Congratulations! You probably have a Debian 13 box now. Enjoy!

There's a chance that if the process failed, but failed late enough in the burn, the U-Boot partition will be fine, and you can actually just try The Easy Way.

## 2.1 (if) You fucked up

If the flash fails, you will see error text in the console pane on the right side of the window. Don't worry! Your device is almost certainly just fine, if it was fine before you started. You just need to:

***WITH THE 12V POWER ADAPTER UNPLUGGED, AND REMAINING UNPLUGGED, THE ENTIRE TIME***
1. open the back panel up with a #2 Phillips head screwdriver (holes have rubber plugs inserted, you also need a thin spudger or flathead to excise them) and a little bit of misery to:
2. get access to the internal programming button to:
3. hold it down (again, it's labelled U-BOOT, and situated beneath the fan assembly), then:
4. Make sure RKDT is open on the "Download Image" screen, and says "Found One MASKROM Device" in the textbox at the bottom.
5. In the memory loader table, you will now need exactly 2 rows. For row #1, set the Path cell value to the path to the miniloader binary file (`rk3568-ddr4-emmc-evb-miniloader.bin`). For row #2, set the Path cell value to the path to the Debian image file. ***IMPORTANT:*** the checkboxes on both rows must be ticked.
6. Verify the checkbox states for row #1 (ticked) and row #2 (also ticked), then click Run.
7. Wait between 3 and 16 minutes for the flashing process to complete.
8. Congratulations! You probably have a Debian 13 box now. Enjoy!
9. If it still didn't work, message me on Reddit. I'll reply when I'm not working.

Same note about Easy Way potentiality exists in the case of FUs here too.

## 3. The Easy Way

Once Armbian is on the box, you do not need this procedure again. To
write a newer image:

***WITH THE 12V POWER ADAPTER UNPLUGGED, AND REMAINING UNPLUGGED, THE ENTIRE TIME***
1. Hold the RESET button (with your implement of choice).
2. Connect a USB A-to-A cable from a PC to the rear USB port. Release the button after
   five seconds, or when your computer notices a new disk is attached.
3. The eMMC appears on the PC as a USB disk of about 29 GB. Write the
   image to it with any raw image writer. Make sure you pick the right
   disk.
4. Disconnect the USB cable, take a deep breath. Bask in it. Really feel how easy that was. It's nice, isn't it?

This works because the U-Boot in this image checks the RESET button at
power-on and then presents the eMMC as an ordinary USB disk. The stock
bootloader's RESET mode is different: it needs RKDT, hence The Medium Way has its own steps above.

## 4. First start

1. Hook it up to an HDMI monitor and a USB keyboard for initial setup.
2. Verify it is powered off. Seriously.
3. Finally, plug the 12V power adapter in.
4. EZPZ

[If you're new, I highly recommend you read the Armbian getting started guide from here.](https://docs.armbian.com/getting-started/first-boot-and-login/)

If you're not new, I highly recommend you run the first-time setup script, then install OMV and call it a day:

[Official Armbian docs OMV install guide.](https://docs.armbian.com/software/omv/)

Or if you want to just rawdog it:

(assuming you're root, otherwise, add sudo)
```
wget -O ~/installomv.sh https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install
nano ~/installomv.sh
```
Make sure it's not malware, then
```
chmod +x ~/installomv.sh
bash ~/installomv.sh
```

### Appendix

† (or go find the Rockchip SPL source and compile it yourself. I compiled it a year ago and forgot where I left the files, sorry)

†† RKDT and drivers are freely available and in the releases page, but I sourced them from [https://docs.radxa.com/en/zero/zero3/low-level-dev/rkdevtool](https://docs.radxa.com/en/zero/zero3/low-level-dev/rkdevtool)
