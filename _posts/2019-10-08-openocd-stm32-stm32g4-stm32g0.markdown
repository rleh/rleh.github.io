---
layout: post
title: "OpenOCD for STM32G0 and STM32G4"
author: Raphael Lehmann
excerpt: "Tutorial to compile OpenOCD yourself with supprot for STM32G0 and STM32G4"
---

Unfortunately the OpenOCD development is not very fast and the latest version is `0.10.0` from January 2017 (October 2019).
This version does not support the STM32G0/STM32G4 as target, nor the ST-Link V3 on e.g. the Nucleo-G474RE board.

Fortunately, it is not so hard to compile OpenOCD yourself, especially under Linux.

#### 1. Clone OpenOCD
Clone the Git Repo:

```
git clone http://openocd.zylin.com/openocd.git
cd openocd
```
#### 2. Apply Patches for STM32G0 and STM32G4

Then additionally checkout the patch from [http://openocd.zylin.com/#/c/4807/](http://openocd.zylin.com/#/c/4807/):
```
git fetch http://openocd.zylin.com/openocd refs/changes/07/4807/4 && git checkout FETCH_HEAD
```

#### 3. Dependencies, Bootstraping, Configuring
Install libraries/dependencies, e.g. on Debian/Ubuntu:

```
apt install make libtool pkg-config autoconf automake texinfo libusb
```

Then run bootstrap and configure scripts:
```
./bootstrap
./configure
```

The scripts inform you about missing libs. Just install the lib and run `./configure` again.

#### 4. Build and Install
Build and install OpenOCD. OpenOCD will get installed to `/usr/lib/`.

```
make
sudo make install
```

#### 5. Uninstall OpenOCD
You probably want to uninstall the OpenOCD package from your distribution:

```
apt remove openocd
```

#### 6. Udev Rules
Copy OpenOCD's new udev file to `/etc/udev/rules.d/` and reload udev rules:

```
sudo cp /usr/local/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d/
sudo udevadm control --reload
```

Have fun programming STM32G0 or STM32G4 microcontrollers with OpenOCD!
