# Install guide

Putting Armbian on an IronCow Zero1 takes two steps. Each has its own
page.

| Step | Page | What you do |
|---|---|---|
| 1 | [BUILD.md](BUILD.md) | Get an image: build your own on a Linux PC, or download the prebuilt one |
| 2 | [FLASHING.md](FLASHING.md) | Write that image to the box's eMMC, replacing the stock firmware |

For step 1 you have a choice. Build your own image with
[BUILD.md](BUILD.md), about half an hour on a fast PC, or download the
prebuilt image from this project's releases page and go straight to
step 2. Building your own means you do not have to trust someone else's
binary. Even if I am a pretty chill guy.

## Before you start

Read the two warnings in the [README](README.md):

- The stock firmware should not be on a network you care about.
- Flashing erases the vendor's serial number. Back it up first if you
  want to keep it (there is a vanishingly low chance that it is a rational thing to want to keep, but, I don't get to make that judgement for you).

## What you need

For step 1:

- A PC running Ubuntu or Debian, or Windows with WSL2.
- Free disk space and an internet connection. [BUILD.md](BUILD.md) has
  the details.

For step 2:

- The box, its power supply, and a way to write the image to it.
  [FLASHING.md](FLASHING.md) lists the cables and tools.

## After the first install

You only do the full procedure once. After that, to write a newer image
("The Easy Way" in [FLASHING.md](FLASHING.md)):

1. Hold the RESET button.
2. Connect a USB A-to-A cable from a PC to the rear USB port. The box
   powers on from the cable; **its 12V adapter must remain unplugged.**
3. The box's eMMC appears on the PC as a USB disk. Write the image to it
   with any raw image writer.
4. Blood pressure = calm. Eased. A smooth 120/80 mm Hg, god willing.
