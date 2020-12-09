# Fairwaves XTRX FPGA images, pre-compiled binaries, and software documentation

This repository hosts FPGA images and pre-compiled binaries for XTRX.

Other sources of documentation:
 - https://github.com/xtrx-sdr/xtrx-docs - for the hardware documentation
 - https://www.crowdsupply.com/fairwaves/xtrx - for general XTRX information

Please report any problems running pre-compiled libraries from this repository directly this repo's [issues](https://github.com/xtrx-sdr/images/issues).

## Structure
 - `binaries` - pre-compiled host binary packages.
 - `sources` - reference source code used for building the binaries. Uses git submodules to reference to the relevant git repositories. See *Manual Compilation* for more details.
 - `firmware` - FPGA images for reflashing. (not yet published)

## Pre-built packages

Binary packages are available for the following distributions. Please note that they are maintained by volunteers and are not officially supported by Fairwaves.

 - [Alpine Linux](https://github.com/aports-ugly/aports/tree/master/ugly/libxtrx)
 - [Arch Linux](https://aur.archlinux.org/packages/?K=xtrx)
 - [Debian](https://packages.debian.org/search?suite=all&searchon=names&keywords=xtrx) (testing/unstable main repo)
 - [openSUSE](https://build.opensuse.org/project/show/hardware:sdr)
 - [Ubuntu PPA](https://launchpad.net/~fairwaves/+archive/ubuntu/xtrx) for Ubuntu 16.04, 18.04, and 19.04

## Manual compilation of host libraries from the `sources` directory

(These instructions were tested under Ubuntu 18.04 and 20.04)

You need C and C++ compiler to build host libraries. `libusb1` is also required if you're using a USB3 adapter. For Debian-based systems:

Ubuntu Dependencies (Does not work with Ubuntu 20.04):
```
% sudo apt-get install build-essential libusb-1.0-0-dev cmake dkms python3 python3-pip gpsd gpsd-clients pps-tools libboost-all-dev git qtbase5-dev libqcustomplot-dev libqcustomplot1.3 libqt5printsupport5 doxygen swig
```
Install Cheetah3 via pip3
```
% pip3 install cheetah3
```
Clone the 'images' repoository to your computer:
```
% git clone https://github.com/xtrx-sdr/images.git
```
Navigate where you downloaded the repo, in this case:
```
% cd images
```
First, you need to clone relevant revisions of source modules:
```
% git submodule init
% git submodule update
```
Now enter the `sources` directory and run:
```
% cd sources
% mkdir -p build
% cd build
% cmake ..
OR if you don't want SoapySDR support:
% cmake -DENABLE_SOAPY=NO ..
AND/OR if you want UDEV rules installed:
% cmake -DINSTALL_UDEV_RULES=ON ..
% make
```
When everything is built you can install:
```
% sudo make install
% sudo ldconfig
```

### PCIe kernel driver compilation

If you're connecting XTRX over a (mini)PCIe bus, you also need to install a kernel driver. 

The easiest way is to use DKMS.

First, make sure DKMS can find the driver sources. `make install` above will install the driver sources into `/usr/local/src` while DKMS expect them in `/usr/src`, so you should create a symlink:
```
% sudo ln -s /usr/local/src/xtrx-0.0.1-2/ /usr/src/
```

And now instruct DKMS to build the module:
```
% sudo /usr/sbin/dkms add -m xtrx -v "0.0.1-2"
% sudo /usr/sbin/dkms build -m xtrx -v "0.0.1-2"
% sudo /usr/sbin/dkms install -m xtrx -v "0.0.1-2"
```

If everything goes well, you can now load the driver:
```
modprobe xtrx
```
and it should appear in `/proc/modules` with relevant messages in `dmesg`.

In case you face any issues with DKMS, [please report the issue](https://github.com/xtrx-sdr/images/issues). Meantime, you should be able to build the driver manually. Enter the `sources/xtrx_linux_pcie_drv` directory and run:
```
% make
% sudo insmod xtrx.ko
```
### Non-root access for /dev/xtrx0 - NOTE: Skip this section if you compiled with UDEV_RULES=ON 

If you want to access XTRX from non-root user (which is the typical case), you need to install `udev` rules. 
```
cp sources/xtrx_linux_pcie_drv/50-xtrx.rules /etc/udev/rules.d/
```
After that, you need to restart `udevd` and remove and insert the XTRX driver (`rmmod xtrx; modprobe xtrx`) or reboot the system. On ubuntu-like systems, you can restart `udev` by
```
udevadm control --reload-rules && udevadm trigger
```


# Flashing XTRX

**WARNING:** Flashing XTRX  has risks of bricking/damaging your XTRX if interupted or a wrong image is flashed. Use Fairwaves USB3 XTRX adapter or Fairwaves PCIe adapter with a compatible JTAG cable to unbrick a bricked XTRX.

The fastest and easiest way to flash XTRX is using `test_xtrxflash` which doesn't require any additional equipment and works both over USB3 and miniPCIe:
```
sudo ./test_xtrxflash -w <image_file.bin>
```
where `<image_file.bin>` is one of the files from the [FPGA folder of this repo](https://github.com/xtrx-sdr/images/tree/master/FPGA):
* `xtrxr4_top.bin` - for XTRX rev4 CS (aka "original" XTRX)
* `xtrxr4PRO_top.bin` - for XTRX rev4 PRO

`test_xtrxflash` relies on a functioning FPGA, so it can't be used for recovery if you brick your XTRX with incorrect flashing. There are two ways to unbrick your XTRX in case of failed flashing:
1. Use Fairwaves USB3 XTRX adapter with Fairwaves fork of [xc3sprog](https://github.com/xtrx-sdr/xc3sprog). Insert XTRX into the adapter and run the following command:
```
$ ./xc3sprog -c usb3380xtrx <image_file.bin>:w:0:BIN
```
2. Use Xilinx USB cable (or any compatible one) with the Fairwaves PCIe adapter.


# Getting started
## Working with XTRX over PCIe bus
PCIe device ID is `10ee:7012`:
```
% sudo lspci -v -d 10ee:
01:00.0 Memory controller: Xilinx Corporation Device 7012
    Subsystem: Xilinx Corporation Device 0007
    Flags: fast devsel, IRQ 255
    Memory at 91510000 (32-bit, non-prefetchable) [disabled] [size=4K]
    Memory at 91500000 (32-bit, non-prefetchable) [disabled] [size=64K]
    Capabilities: [40] Power Management version 3
    Capabilities: [48] MSI: Enable- Count=1/4 Maskable- 64bit+
    Capabilities: [60] Express Endpoint, MSI 03
    Capabilities: [100] Device Serial Number 00-00-00-00-12-34-56-78
```

If driver `xtrx` is loaded correctly, a character device `/dev/xtrx0` should appear.


## Working with the USB3 adapter
USB3 adapter should appear on your system as USB device `0525:3380`. Check it by running `lsusb`:
```
% lsusb
Bus 002 Device 002: ID 0525:3380 Netchip Technology, Inc. 
```

## Testing if XTRX is alive
Navigate to: 
```
% cd /usr/local/lib/xtrx/
```
Run this command to check that your XTRX is connected properly and is not dead:
```
% sudo ./test_xtrx -t -l2
```
The output should be like this
```
CPU Features: SSE2+ SSE4.1+ AVX+ FMA+
Master: 31999999.863761; RX rate: 3999999.982970; TX rate: 0.000000
RX tunned: 899999999.046326
RX bandwidth: 2500000.000000
RX LNA gain: 15.000000
RX SAMPLES=524288 SLICE=524288 PARTS=1
PROCESSED RX SLICE 0 /0: res 0 TS:       0      65746 us DELTA    118 us LATE    328 us 524288 samples R=    ffdff6ffeffd DELTA=-1553528470
XXXXX Overruns:0 Zeros:0
xAND=[0xffc0,0xffc0,0xff40,0xffc0] xOR=[0xffff,0xffff,0xff7f,0xffdf]
Success!
Processed RX 2 x 3.984 = 7.969 MSPS (WIRE: 31.874542)    TX 2 x 0.000 = 0.000 MSPS (WIRE: 0.000000 MB/s) 
```

If you see output similar to the one above, XTRX is ready to rock! And you can now build your very own application or install one already coming with the XTRX support:

 - gr-osmosdr and GNURadio: https://github.com/osmocom/gr-osmosdr NOTE: Compile it from source only!!! *Thanks to dchard for the patch and gr-osmosdr PR*
 - sdrangel: https://github.com/xtrx-sdr/sdrangel
 - gqrx (no special branch, but install XTRX branch of gr-osmosdr first): http://gqrx.dk/
 - kalibrate: https://github.com/xtrx-sdr/kalibrate-rtl
 - osmo-trx: https://github.com/xtrx-sdr/osmo-trx
 - dump1090 (via SoapySDR): https://github.com/bkerler/dump1090 


# FAQ
## I don't see XTRX in `lspci` output
If XTRX is installed into a miniPCIe slot and an LED near the edge of XTRX is blinking with the ON-OFF-ON-OFF-.. pattern with ~3sec period,  it means XTRX hasn't been enumerated on the PCIe bus.

Make sure that the miniPCIe slot has PCIe lanes routed. Some systems use miniPCIe slots to connect USB-only devices and do not route PCIe lanes of the miniPCIe slot. In this case, you can try XTRX with other miniPCIe slots.

In all other cases, feel free to [contact us](https://xtrx.io/contact/).

## Which board/laptop can I use with XTRX in PCIe mode?
**You can use any board/laptop with XTRX with a USB3 adapter. Below assumes that you want to use XTRX in PCIe mode**

Generally, your board should have the miniPCIe socket **with the PCI lane**. Most laptops don't work, since generally they have PCI lane only on half-sized miniPCIe sockets (used for WiFi devices) while full-sized sockets designed mostly for use with SATA and USB lanes only. Industrial SBCs frequently have at least one such socket thus are more suitable for XTRX.

These devices were tested by our customers or us and were reported working fine with XTRX:
 - [UpSquared](https://up-board.org/upsquared/specifications/) by UpBoard
 - [Jetson TK1](https://www.nvidia.com/object/jetson-tk1-embedded-dev-kit.html) by NVidia
 - various devices by [LEX](http://www.lex.com.tw/), specifically [TERA + 2I610DW](http://www.lex.com.tw/products/TERA-2I610DW.html) and [3I610CW](http://www.lex.com.tw/products/3I610CW.html)
 - various SBCs by [PC Engines](https://www.pcengines.ch/), specifically [apu2c4](https://pcengines.ch/apu2c4.htm), [apu1d4](https://www.pcengines.ch/apu1d4.htm), [apu2d4](https://pcengines.ch/apu2d4.htm), and [apu4b4](https://www.pcengines.ch/apu4b4.htm) (may be suitable for 1 XTRX only, check the specs)
 - [Newport GW6300](http://www.gateworks.com/product/item/newport-gw6300-single-board-computer) by GATEWORKS
 - [ROCKPro64](https://www.pine64.org/rockpro64/) by Pine64
 - [IPC3](https://www.fit-pc.com/web/products/ipc3/) with [FM-XTDM2](http://www.fit-pc.com/wiki/index.php/FACE_Modules:FM-XTDM2) module by fit-PC
 - [ARTiGO A1200](https://www.viatech.com/en/support/eol/artigo-a1200-eol/) by VIA (see #28)

These SBCs might work with XTRX, but we haven't tested them. Try at your own risk!
 - various SBCs by [AAEON](https://www.aaeon.com/en/c/embedded-single-board-computers/)
 - various SBCs by [Advantech devices](https://www.advantech.com/products/embedded-single-board-computers/sub_c427052f-f55f-4d47-9a84-3a11ed5295df)
 - various devices by [Logic Supply](https://www.logicsupply.com/)
 - [MPT-7000V](https://www.ibase.com.tw/english/ProductDetail/IntelligentTransportation/MPT-7000V) by IBASE; suitable for 1x XTRX only
 - [Nuvo-5100VTC](https://www.neousys-tech.com/en/product/application/in-vehicle-computing/nuvo-5100vtc); seems suitable for 2x XTRX (4x miniPCIe sockets, 2x of them supports USB signals only)
 - [ECS-9110](http://www.vecow.com/dispPageBox/vecow/VecowCT.aspx?ddsPageID=PRODUCTDTL_EN&dbid=4357925395) by Vecow most likely would work
 - [EC500-KH](https://www.dfi.com/Product/Index/1412) and [EC70B-SU](https://www.dfi.com/Product/Index/125) by DFI most likely would work
 - some older versions of Intel NUC which have miniPCIe may work
 - most likely Dell Latitude D420 would work; it has full-sized miniPCIe (used for WiFi by default) and no PCI whitelist in BIOS
 - some outdated EeePCs like 901 and 701 by ASUS may work (assuming based on the same premises as for Dell Latitude D420)
 - any device with full-featured PCIe x2 socket or USB3+ port if you use XTRX with PCIe x2 Front End or USB 3 Adapter respectively (see [CrowdSupply page](https://www.crowdsupply.com/fairwaves/xtrx) for details and order)
You can also check other SBCs by mentioned vendors.

## XTRX installed into a miniPCIe slot prevents the system from booting (don't even get to BIOS)
This problem can happen if your motherboard provides SMB signals to the miniPCIe connector. Most systems don't. The XTRX rev.4 use reserved pins that are held in hi-Z state. However, during the startup, DC/DC chip isn't initialized, and XTRX is pulling down the SMB DATA line, which is often connected to chips critical for the system booting. We're working on a firmware fix, but currently, hardware modification (cutting the fuse on the bottom side of XTRX near the edge as pictured below) is the only solution.
Cut the fuse pictured below with a sharp knife. Make sure you do not use much pressure, or you can accidentally cut other traces.

![smb_fuse](https://xtrx-sdr.github.io/images/smb_data_fix.jpg)

Systems that known to have such problem

| Vendor   | Model        | Comments |
| -------- | ------------ | -------- |
|  Jetway  | NP591        |   |
|  Via     | Artigo A1200 |   |
|  Intel   | DQ77KB       | SMBus even mentioned in the block diagram [doc](https://www.intel.com/content/dam/support/us/en/documents/motherboards/desktop/dq77kb/dq77kb_techprodspec08.pdf) |
|  Lex     | 2I610HW      | Both miniPCIe have SMBus routed |


## I connected XTRX directly via micro-USB cable, and I don't see it `lsusb` 
Unfortunately, USB2 PHY software isn't ready yet. Check issue [#9](https://github.com/xtrx-sdr/images/issues/9) for more details and suggestions.

## Is Windows driver support planned?
We plan to add Windows PCIe drivers, but we don't have a specific schedule for this right now. If it's blocking factor, please [contact us](https://xtrx.io/contact/). However, all other software layers are windows compatible, and USB3380 adapter should work via WinUSB.

## I've got a DMA buffer issue on my ARM device
You should probably increase DMA coherent pool using `coherent_pool=32M` kernel option in case you see the following lines in the `dmesg` output:
```
[ 7.171608] xtrx: Failed to allocate 31 DMA buffer
[ 7.171632] xtrx 0000:01:00.0: Failed to register TX DMA buffers.
```
For the details, see issue [#37](https://github.com/xtrx-sdr/images/issues/37).

## How to use XTRX GPS to receive NMEA data and PPS signal? (Works on Ubuntu 18.04.)

Prerequisites:
1. You should connect XTRX with miniPCIe or PCIe since USB3 libs [don't expose GPS interface](#can-xtrx-gps-be-accessed-through-usb3) to the system.
2. Make sure that XTRX PCIe drivers are installed. You don't need XTRX libraries since it solely depends on kernel devices and `gpsd`/`pps-tools` software.

To receive GPS NMEA data like current time and geoposition:
1. Connect the GPS antenna to XTRX and ensure there is clear sky visibility to acquire GPS signal. Note that coated glass windows may completely block GPS signal, so we recommend putting the GPS antenna outside of a window if you're testing indoors.
2. Install `gpsd` and `gpsd-clients` packages (for example, for Ubuntu: `apt-get install gpsd gpsd-clients`).
3. Run `gpsd` with `/dev/ttyXTRX0` specified as the device. In Debian-based distros you can specify device path in `/etc/default/gpsd` file and restart `gpsd` service). Otherwise, you can run it with device path as the last CLI argument: `sudo gpsd -D 5 -N -n /dev/ttyXTRX0`.
4. Run `cgps` to see the NMEA data coming from the XTRX GPS unit, as well as it's decoding into a date, time, and geopositioning.

To test PPS (aka [Pulse-per-second signal](https://en.wikipedia.org/wiki/Pulse-per-second_signal)):
1. Connect the antenna as described for the NMEA testing above.
2. Install `pps-tools`.
3. Run `sudo ppstest /dev/pps0`. Note, if you have more than one PPS device, you can find correct one using, for an instance, `sudo dmesg | grep xtrx_pps` command, just take `ppsN` from the output where `N` is a number, and use it as device's filename in the `/dev` synthetic file system. On successful receiving of PPS signal, you'll see lines like this: `source 0 - assert 1560521435.029030769, sequence: 615 - clear  0.000000000, sequence: 0`.

## Can I access XTRX GPS through USB3?
Currently, XTRX GPS can be accessed only through PCIe. There are only four USB endpoints available with the USB3 adapter, and SDR part of XTRX uses two of them. So only two USB endpoints are available while USB serial normally requires three USB endpoints. This can potentially be resolved by writing a custom kernel driver - contributions are welcome if someone wants to do that!
