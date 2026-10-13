# Building the image

This page takes you from nothing to an Armbian image file for the
IronCow Zero1. It assumes you can use a Linux command line. You do not
need to have used Armbian's build system before.

## How it fits together

Armbian's build system is a set of scripts in one git repository,
[armbian/build](https://github.com/armbian/build). It knows about several
hundred boards. This box is not one of them.

The build system reads extra board definitions from a folder called
`userpatches` inside its own tree. This repository's `userpatches` folder
contains everything needed to teach it about the IronCow Zero1. You copy
that folder in, then ask for a build of the board `ironcow-zero1`.

## What you need

- A 64-bit PC running Ubuntu 24.04. Other recent Ubuntu and Debian
  releases, and WSL2 on Windows, generally work; this was developed on
  Ubuntu 24.04 under WSL2.
- About 50 GB of free disk space.
- An internet connection. The first build downloads several gigabytes.
- `sudo` rights. The build asks for your password.
- `git`.

On WSL2, keep the build inside the Linux filesystem (somewhere under
your home folder), not under `/mnt/c`. It is far faster and avoids file
permission problems.

## 1. Get Armbian's build system

```
cd ~
git clone --depth=1 https://github.com/armbian/build armbian-build
```

## 2. Get this repository

```
cd ~
git clone https://github.com/gloriouspidgeon855/ironcow-zero1-complete-armbian-candidate.git ironcow-zero1
```

## 3. Copy `userpatches` into the build tree

```
cp -r ~/ironcow-zero1/userpatches ~/armbian-build/
```

Check that the board file landed in the right place:

```
ls ~/armbian-build/userpatches/config/boards/
```

It must list `ironcow-zero1.conf`. If the folder is somewhere else, for
example `userpatches/userpatches/`, the build will not find the board.

## 4. Build

```
cd ~/armbian-build
./compile.sh build BOARD=ironcow-zero1 BRANCH=current RELEASE=trixie BUILD_MINIMAL=yes BUILD_DESKTOP=no KERNEL_CONFIGURE=no PREFER_DOCKER=no
```

| Option | Meaning |
|---|---|
| `BOARD=ironcow-zero1` | The board defined by this repository |
| `BRANCH=current` | Armbian's current kernel line, 6.18 at the time of writing |
| `RELEASE=trixie` | Debian 13 |
| `BUILD_MINIMAL=yes`, `BUILD_DESKTOP=no` | A small command-line system, as suits a NAS |
| `KERNEL_CONFIGURE=no` | Use Armbian's kernel configuration without asking |
| `PREFER_DOCKER=no` | Build directly on the PC, not in a container |

The first build takes the longest, because it downloads the kernel and
U-Boot sources and compiles everything. On a 16-core desktop it took
about 30 minutes; later builds took about 15.

## 5. Find the image

```
ls ~/armbian-build/output/images/
```

The image is the `.img` file, named like this:

```
Armbian-unofficial_26.11.0-trunk_Ironcow-zero1_trixie_current_6.18.56_minimal.img
```

The version numbers will differ as Armbian and the kernel move on. The
image is about 2 GB. The `.sha` file next to it holds its checksum.

## 6. Check the build (optional)

The one thing that must be right is that the kernel package contains
this board's device tree. To confirm it:

```
dpkg-deb -c ~/armbian-build/output/debs/linux-dtb-current-rockchip64_*.deb | grep ironcow
```

It must print a line ending in `rk3568-ironcow-zero1.dtb`. If it prints
nothing, the `userpatches` folder is in the wrong place; go back to
step 3.

## What the `userpatches` folder contains

| File | Purpose |
|---|---|
| `config/boards/ironcow-zero1.conf` | The board definition: which kernel and U-Boot to use, extra packages, and a few build steps |
| `kernel/archive/rockchip64-6.18/dt/rk3568-ironcow-zero1.dts` | The device tree: the description of the hardware that the kernel reads |
| `u-boot/v2026.07/defconfig/ironcow-zero1-rk3568_defconfig` | The U-Boot configuration |
| `u-boot/v2026.07/dt_upstream_rockchip/rk3568-ironcow-zero1.dts` | The same device tree, for U-Boot. Keep the two copies identical |
| `u-boot/v2026.07/dt_uboot/rk3568-ironcow-zero1-u-boot.dtsi` | U-Boot-only additions to the device tree |
| `bootenv/rk35xx.txt` | Boot settings: kernel messages on the serial port |

## Choices built into the image

These are decisions made for this repository. Change them in
`config/boards/ironcow-zero1.conf` before building if you disagree.

- **Kernel packages are held.** The line `BSPFREEZE="yes"` stops
  `apt upgrade` from replacing the kernel with Armbian's own, which does
  not carry this board's device tree. The [README](README.md) explains.
- **Extra packages.** `PACKAGE_LIST_BOARD` adds tools useful on a NAS,
  such as `smartmontools`, `mdadm` and `hdparm`.
- **No `fake-hwclock`.** The box has a real battery-backed clock, and
  that package would override it at every start.
- **Verbose start-up.** `bootenv/rk35xx.txt` sends kernel messages to
  the serial port. Delete that file for Armbian's quieter default.

## If the build fails

- **The build does not know the board `ironcow-zero1`.** The
  `userpatches` folder is not where step 3 puts it.
- **The kernel builds, but step 6 finds no device tree.** Armbian's
  `current` kernel has moved to a newer version than 6.18. The device
  tree folder is named for the kernel version, so it is no longer read.
  Rename `kernel/archive/rockchip64-6.18` to match the folder name for
  the new version under `patch/kernel/archive/` in Armbian's tree. The
  device tree may then need changes for the new kernel.
- **U-Boot fails to build.** This repository was last built against
  Armbian's `main` branch in October 2026. A later Armbian may need the
  `u-boot/v2026.07` folder and the two version lines in the board file
  moved to a newer U-Boot.

Armbian adds a few example files and empty folders of its own to
`userpatches` on every build. That is normal.

## Next step

You now have an image file. Go to **[FLASHING.md](FLASHING.md)** to put
it on the box.
