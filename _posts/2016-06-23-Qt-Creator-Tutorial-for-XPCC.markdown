---
layout: post
title:  "How to use QT Creator for XPCC development"
date:   2016-06-23 21:30:00
categories: xpcc development
description: This tutorial shows how to adapt the Qt Creator IDE for developing and comfortable debugging XPCC projects
---

This tutorial shows how to adapt the Qt Creator IDE for developing and comfortable debugging XPCC projects on STM32 microcontrollers.

In this tutorial I use xpccs *blink* example project for the STM32F072B Discovery board.
If you want to use a STM32F3 or STM32F4 microcontroller, create a *OpenOCD provider*, *Device* and *Kit* for each microcontroller family.

### Installation

Install Qt creator version 4.0 or above and the GCC ARM Embedded toolchain including GDB.

```
sudo dnf install qtcreator arm-none-eabi-gcc arm-none-eabi-gdb
```

### Global Qt creator setup

#### Enable Bare Metal plugin

*Help* -> *About Plugins...*

![Enable BareMetal plugin](/assets/images/qt-creator-tutorial/enable-baremetal-plugin.png)

Enable the BareMetal plugin and restart Qt creator.

#### Code Style
Go to *Options* dialog: *Tools* -> *Options...*

Go to Tab *C++*, create a copy of the `Qt [builtin]` code style, name it `XPCC` and click *Edit...*.

![Code Style](/assets/images/qt-creator-tutorial/code-style.png)

Set *Tab policy* to `Tabs only`, save and exit.

#### GDB and OpenOCD global configuration

In *Build & Run* and sub-tab *Debuggers* click *Add* to add the `arm-none-eabi-gdb` debugger to Qt creator:
![Create new `arm-none-eabi-gdb` debugger](/assets/images/qt-creator-tutorial/add-arm-none-eabi-gdb.png)

Go to tab *Bare Metal* and select *Add* -> *OpenOCD*.
![Add new OpenOCD GDB server provider](/assets/images/qt-creator-tutorial/add-openocd-provider.png)

Use the following settings:
![OpenOCD GDB server provider settings](/assets/images/qt-creator-tutorial/openocd-provider-settings.png)
*Configuration file* is `/usr/share/openocd/scripts/board/stm32f0discovery.cfg` for STM32F0 (on Fedora 23).

For STM32F3 use `/usr/share/openocd/scripts/board/stm32f3discovery.cfg`, for STM32F4 `/usr/share/openocd/scripts/board/stm32f4discovery.cfg`.

For the next step go to the *Devices* tab and *Add...* a new *Bare Metal Device*

![Add a new device](/assets/images/qt-creator-tutorial/add-bare-metal-device.png)

Use the following settings:

![Qt creator device settings](/assets/images/qt-creator-tutorial/bare-metal-device-settings.png)

Again, go to tab *Build & Run*, sub-tab *Kits* and click *Add* to add a new so called Kit.

![Settings for the STM32F0 Kit](/assets/images/qt-creator-tutorial/stm32f0-kit-settings.png)

* Device type: *Bare Metal Device*
* Device: *STM32F0* (the device we created previously)
* Do not care about the compiler
* Debugger: *ARM none EABI GDB*
* Qt version: *None*

Click *Ok*.

### XPCC project setup

Change to project folder: `cd src/RCA/2016/xpcc/examples/stm32f072_discovery/blink/`

Run `scons qtcreator` to generate the Qt creator project files fpr the xpcc project.

```
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
Template: '/home/raphael/src/RCA/2016/xpcc/templates/qtcreator/project.creator.in' to 'blink.creator'
Template: '/home/raphael/src/RCA/2016/xpcc/templates/qtcreator/project.config.in' to 'blink.config'
Template: '/home/raphael/src/RCA/2016/xpcc/templates/qtcreator/project.files.in' to 'blink.files'
Template: '/home/raphael/src/RCA/2016/xpcc/templates/qtcreator/project.includes.in' to 'blink.includes'
scons: done building targets.
```

Open project with Qt creator: *File* -> *Open File or Project (Ctrl-O)*

![Select `*.creator` file](/assets/images/qt-creator-tutorial/open-creator-file.png)

![The view should be like this](/assets/images/qt-creator-tutorial/qtcreator-1.png)

Select the *Projects* view from the left side.

First, click *Add Kit* -> *STM32F0*, then remove the by default created *Desktop* using the icon to the right of *Desktop* -> *Remove Kit*.
![Remove the desktop kit](/assets/images/qt-creator-tutorial/remove-desktop-kit.png)

Next go to the *Build Settings* and remove all existing *Build Steps* and *Clean Steps* and add a new *Custom Process Step* build step with the command `scons` and argument `program`.

Likewise add a *Clean Step* using a *Custon Process Step* with command `scons` and argument `-c`.

![Qt creator project build settings](/assets/images/qt-creator-tutorial/project-build-settings.png)

Switch to *Run Settings* and select *Run on GDB server or hardware debugger* as *Run Configuration*. Select the **.elf*-file as executable.
![Qt creator project run settings](/assets/images/qt-creator-tutorial/project-run-settings.png)

Now you can compile, program and debug your xpcc application comfortably in Qt creator.
