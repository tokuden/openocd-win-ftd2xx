[日本語版の説明はこちら](./READMEj.md)
# Welcome to OpenOCD-win-ftd2xx

OpenOCD-win-ftd2xx is a build of OpenOCD to run on Windows.
It has been rewritten to work with FTDI's proprietary FTD2XX driver instead of the open source libusb or libftdi.

## Features
* Runs on Microsoft Windows (no virtual environment required)
* Customized to run on the FTDI FTD2XX driver, so *no driver swapping is required with Zadig*.
* Can be used with any USB-JTAG device using MPSSE, such as the FTDI FT232H, FT2232D/H, and FT4232H
* Based on the latest OpenOCD 0.12

We have confirmed that it can connect to ZYNQ7000, UltraScale+, and Raspberry Pi 4.
It runs as a native Windows app, so no virtual environment or containers such as WSL are required.
You don't have to worry about annoying people who aren't familiar with it by asking them to prepare WSL.

## Download and Install
Download and unzip [OpenOCD-Win-FTD2XX.zip](/tokuden/openocd-win-ftd2xx/OpenOCD-Win-FTD2XX.zip) .

This repository is a fork of the original OpenOCD on August 14, 2025, so it contains a variety of files, but the only files users truly need to run the project are those contained in OpenOCD-Win-FTD2XX.zip.

![folder after unzipping](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/folder.png)

## How to use
Open MS-DOS prompt and type such as.

`openocd.exe -f ft2232h.cfg -f zynq7000.cfg`

We prepare the following config files.

* np1167.cfg - Configuration file for the interface using the Tokuden's NP1167A/B cable
* ft2232h.cfg - Generic interface config file to use FT2232H MPSSE
* TE0802.cfg - Interface config file for TrenzElectronic's TE0802
* zynq7000.cfg - Target config file for ZYNQ7000
* xilinx_zynqmp.cfg - Target config file for ZYNQ UltraScale+

We recommend make a batch file when you decide 
Once you have decided on the configuration, it is better to create a batch file.  
For example, if using [Tokuden's NP1167A/B cable](https://mitoujtag.jp/mpssejtag.html), specify `-f np1167.cfg` .

When you start it, the following screen will appear.

![Launch screenshot OpenOCDWin](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/openocdwin.png)

To debug the CPU, you will need to log in to localhost:4444 with TELNET or connect remotely with GDB.
For example, to dump the contents of physical memory address 0, use TeraTerm to connect to localhost:4444 with TELNET and type the following commands.

```
targets
halt
mdw phys 0 0x100
redume
```

If you halt, the CPU will remain stopped unless you resume.

![Memory dump](https://github.com/tokuden/openocd-win-ftd2xx/blob/images/dump.png)

## The main feature of OpenOCD-win-FTD2XX
The biggest feature of the Nahitafu version of OpenOCD is that it runs on Windows using FTDI's original device driver without using libusb or WinUSB.

FTDI device drivers are automatically downloaded and ready to use the moment they are plugged into a Windows PC, so there is no need to install a device driver as long as you use them normally. They are excellent device driverss that can be used immediately after plugging them in.

However, OpenOCD is open source software and is licensed under the GPL.
The problem is that GPL enthusiasts have the crazy idea that "GPL software is righteous, so proprietary software (software whose source code is not publicly available) should not be linked to it," which has led people to use open source device drivers created by volunteers such as libusb and libftdi1 instead of the original FTDI libraries. I think the FTDI-related code was removed from OpenOCD in v0.10.0-rc1.

However, Windows can only use one driver per USB ID.

Therefore, they recommend using a dangerous tool called Zadig to modify the registry so that libusb is used instead of the FTDI device driver.

If you use zadig to replace the FTDI driver with libusb, the FTDI driver will no longer be usable on your PC, so you will have to make the big sacrifice of not being able to use other apps that use FTDI.
To resolve this situation that the FTDI driver is no longer usable, you will need to uninstall and reinstall the device driver, but if you do this repeatedly, Windows will gradually break down.

The reason OpenOCD doesn't use FTDI drivers is to avoid mixing open source and proprietary drivers, but this doesn't bring any benefits to users. Rather there is only a risk of having to swap drivers every time.
So I, Nahitafu, rewrote mpsse.c, which was written to use the FTDI genuine drivers instead of libusb and libftdi.

## Build information
I modified only [src/jtag/drivers/mpsse.c](/tokuden/openocd-win-ftd2xx/blob/master/src/jtag/drivers/mpsse.c) from original OpenOCD 0.12.

configure options are 

```
./configure --disable-werror --enable-stlink=no --enable-ti-icdi=no --enable-ulink=no --enable-angie=no --enable-usb-blaster-2=no --enable-ft232r=no --enable-vsllink=no --enable-xds110=no --enable-osbdm=no --enable-opendous=no --enable-armjtagew=no --enable-rlink=no --enable-usbprog=no --enable-esp-usb-jtag=no --enable-cmsis-dap-v2=no --enable-cmsis-dap=no --enable-nulink=no --enable-kitprog=no --enable-usb-blaster=no --enable-presto=no --enable-openjtag=no --enable-linuxgpiod=no --enable-dmem=no --enable-sysfsgpio=no --enable-remote-bitbang=no --enable-linuxspidev=no --enable-buspirate=no --enable-dummy=no --enable-vdebug=no --enable-jtag-dpi=no --enable-jtag-vpi=no --enable-rshim=no --enable-xlnx-xvc=no --enable-jlink=no --enable-parport=no --enable-amtjtagaccel=no --enable-ep93xx=no --enable-at91rm9200=no --enable-bcm2835gpio=no --enable-imx-gpio=no --enable-am335xgpio=no LDFLAGS="-lftd2xx"
```

There are many options. These remove unnecessary cables.

Important options are only `--disable-werror LDFLAGS="-lftd2xx" `.
This executable can be complied under the MSYS2 MINGW64.
