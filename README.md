# IronCow Zero1 board support for Armbian

Armbian board support for the IronCow Zero1 (铁牛Zero1), a two-bay NAS
built on the Rockchip RK3568. This repository holds an Armbian
`userpatches` folder: the board configuration, the device tree and a
mainline U-Boot configuration. With it, the standard Armbian build
system produces a Debian 13 image for this box.

This is a release candidate. It is not part of Armbian, and Armbian does
not support it.

## Start here

**[INSTALL-GUIDE.md](INSTALL-GUIDE.md)** walks through the whole job:
building the image and putting it on the box.

***Read this page in its entirety before you flash anything.***

## What is in this repository

| Path | What it is |
|---|---|
| [INSTALL-GUIDE.md](INSTALL-GUIDE.md) | Where to start |
| [BUILD.md](BUILD.md) | How to build the image |
| [FLASHING.md](FLASHING.md) | How to put the image on a stock unit |
| `userpatches/` | The files Armbian's build system reads; see [BUILD.md](BUILD.md) |

## Warning: keep the stock firmware off any network you care about

Let me be serious for a second.

The firmware this box ships with is not safe to run, and this is not a
matter of a few missing patches, or issues to be fixed remotely. Based on my review of the vendor's 
software, the device is built to be operated remotely by someone other
than you. I have found no reason to believe that the manufacturer built it this way maliciously, or uses their systems for any hostile purpose. However, it takes instructions from outside your home. It does not meaningfully check who is giving them, or what it is being told to
do. The protections in front of its own login are weak enough to be decorative. None of this asks your permission, and there may be little or no warning if something goes wrong. I find the software quality on these devices to be unconscionable for a personal network. That is why I am publishing this clean build target: to provide an alternative.

Treat a stock unit as a machine that already belongs to someone else:

- Do not connect it to a network at all if you can avoid it.
- If you must, give it an isolated network of its own. Never let it
  onto the network your computers, phones and other devices share.
- Do not put anything on it that you would mind losing or leaking.
- Do not expect the device to stay the same if you leave it on the internet for a long period of time.
- Do not give it a password you use anywhere else.
- Do not give it as a gift to anyone. Don't expose anyone to it without informing them that it can not be trusted with sensitive data or sensitive access.

A device like this is what people who build botnets go looking for:
always on, full of your files, sitting inside your network, and willing
to follow orders silently, with no warning, and no recourse after the fact. If you modify or replace the stock firmware on your unit, I will not help you restore the factory software.

## Kernel packages are held

Images built from this repository mark the Armbian packages they install
(kernel, device tree, U-Boot, board support) as held, so `apt upgrade`
leaves them alone. The reason: the kernel and device tree packages are
shared by every rockchip64 board, and the ones in Armbian's repository do
not contain this board's device tree. Installing them would leave the box
unable to boot.

Everything else upgrades normally. To see what is held:

    apt-mark showhold

Once this board is part of Armbian itself, reflash an official image, or
release the holds with `apt-mark unhold` on each package listed.

## What works

Tested on the hardware. Most rows were checked on the current image
(Armbian 26.11 trunk, Linux 6.18, U-Boot v2026.07). CPU scaling,
throttling and network speed were last measured two builds earlier.

| Function | State |
|---|---|
| Boot from eMMC, serial console | Works |
| Both SATA bays, SMART | Works |
| 2.5 Gbit/s Ethernet, stable MAC address | Works |
| Rear USB 3.0 port, USB Type-C port (USB 2.0) | Works |
| HDMI video and audio | Works |
| Real-time clock with battery | Works |
| Front power button, rear RESET button | Works |
| CPU frequency and voltage scaling, thermal throttling | Works |
| Fan speed control | Works, see below |
| Drive bay and power LEDs | Works, but disk lights flash together |
| Reflash through the RESET button, case closed | Works |
| Buzzer | Described but switched off on purpose |

Not tested: booting from USB, LAN, or SATA and suspend/sleep. In theory, all of these should work. Let me know if they don't.

Things to know:

- **The fan is not automatic.** It cools the drive bays, not the
  processor, so the kernel does not drive it from the processor
  temperature. It starts at 70% and stays there until something writes a
  new value (0 to 255) to
  `/sys/devices/platform/pwm-fan/hwmon/hwmon*/pwm1`.
- **RESET does two jobs.** A short press while running restarts the box.
  Held while power is applied, it turns the eMMC into a USB disk on the
  rear port, so you can write a new image without opening the case.
- **The fan header and the drive bays stay powered when the box is off.**
  That is just... how the board is built. A four-wire PWM fan in place of the
  stock fan will run at full speed while the box is off.

## Fair Warning: If you want to, back up the factory serial number and MAC

Installing this image overwrites the area of the eMMC where the vendor
keeps the unit's serial number and Ethernet MAC address. Nothing in this
project preserves or restores it. Armbian does not need it: the network
port gets a stable MAC address derived from the SoC's own serial.

You only need the backup if you ever want to return to the vendor
firmware, which this project recommends against (see the warning above).

The vendor storage area holds the only identity the vendor gave this
unit: its serial number and its MAC address. The MAC address is also
printed on a sticker on the bottom of the device, so it is not at risk.
The serial number is not printed anywhere on the device. If you
overwrite the eMMC without backing it up, it is gone for good, and the
vendor firmware cannot be updated without it.

Whether that matters is up to you. The stock firmware does not work
without internet access, and I strongly advise against ever
connecting it to a network, so in my view, you lose nothing of value, and gain greatly with safe, open software running on it.

### Getting a root shell on the stock firmware

The backup needs root on the stock firmware. The method here is
physical: it uses the internal serial console, so you have to open the
case. It works only with the box in your hands and allows no remote access.

1. Open the case. Connect a 3.3 V USB-to-serial adapter to the internal
   UART header. The console runs at 1500000 baud, 8N1.
2. Apply power and watch the console. When the bootloader shows its
   countdown ("Hit any key to stop autoboot"), press and hold down Ctrl-C to stop at
   the `=>` prompt.
3. Tell the kernel to start a shell in place of the normal startup, then
   boot:

        setenv bootargs ${bootargs} init=/bin/sh selinux=0
        boot

4. The box starts to a root shell with no password, shown as a bare `#`.

The change is not saved. It lasts only for this session, because nothing
is written to the bootloader's stored environment, so the next normal
start is unaffected.

### Reading the serial number

This shell is a stripped BusyBox with almost nothing set up, but the
internal eMMC (`/dev/mmcblk0`) is already present, and reading the serial
number needs nothing more.

The vendor keeps the serial number near the start of the vendor storage
area, as plain text after the tag `DVKR`. Print that area to the console
and read the number off the screen:

    dd if=/dev/mmcblk0 bs=512 skip=7168 count=2 2>/dev/null | tr -cd '[:print:]\n'; echo

Write down the 32-character serial number. That is the only identity not
printed on the box; the MAC address is on the sticker underneath, and is
its last 12 characters. If you want to preserve what is unique about your unit, that's all you have to save.

### I Will Not Help You Make A Full Binary Backup

Again: The vendor-provided software is not safe, and I do not find it conscionable to use for any persistent active task of any kind. I have not been able to come up with any way to pacify it. Therefore, I will not help you make a backup of the stock image, because I strongly believe you should flee from it with as much haste as you can make. If you value the ability to restore to a stock-like state, you will need to work through the process on your own.
