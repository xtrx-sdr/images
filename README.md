# XTRX FPGA images & pre-compiled binaries

This repository hosts FPGA images and pre-compiled binaries for XTRX. Only rev4 hardware and Ubuntu 16.04 x86_64 binaries are published at this moment, that works well on Ubuntu 18.04 x86_64 as well.

NOTE: rev3 hardware support is phasing out, please contact <xtrx@fairwaves.co> in order to get support.

Please report any problems running pre-compiled libraries from this repository directly this repo's [issues](https://github.com/xtrx-sdr/images/issues).

## Structure
 - `binaries` - pre-compiled host binary packages.
 - `sources` - reference source code used for building the binaries. Uses git submodules to reference to the relevant git repositories. See *Manual Compilation* for more details.
 - `firmware` - FPGA images for reflashing. (not yet published)

## Pre-built packages

 - [Alpine Linux](https://github.com/aports-ugly/aports/tree/master/ugly/libxtrx)
 - [Arch Linux](https://aur.archlinux.org/packages/?K=xtrx)
 - [openSUSE](https://build.opensuse.org/repositories/hardware:sdr)
 - [Ubuntu 18.04](https://github.com/satunnainen/images/tree/master/binaries/Ubuntu_18.04_amd64)

## Manual compilation of host libraries from the `sources` directory

You need C and C++ compiler to build host libraries. `libusb1` is also required if you're using a USB3 adapter. For Debian-based systems:
```
% sudo apt-get install build-essential libusb-1.0-0-dev cmake dkms python-cheetah python
``` 
For Ubuntu 18.04 system you need to install Python 2.7 since python-cheetah is only available for 2.7

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
% make
```
When everything is built you can install them:
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
### Non root access for /dev/xtrx0

If you want to access XTRX from non-root user (which is the typical case) you need to install `udev` rules. 
```
cp sources/xtrx_linux_pcie_drv/50-xtrx.rules /etc/udev/rules.d/
```
After that you need to restart `udevd` and remove and insert the XTRX driver (`rmmod xtrx; modprobe xtrx`) or reboot the system. On ubuntu-like systems you can restart `udev` by
```
udevadm control --reload-rules && udevadm trigger
```

# Getting started
## Working with XTRX over PCIe bus
PCIe device ID is `10ee:7012`:
```
% lspci -v -d 10ee:
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
Run this command to check that your XTRX is connected properly and is not dead:
```
% test_xtrx -t -l2
```

Output should be like this
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

 - GNU Radio: https://github.com/xtrx-sdr/gr-osmosdr
 - sdrangel: https://github.com/xtrx-sdr/sdrangel
 - gqrx (no special branch, but install XTRX branch of gr-osmosdr first): http://gqrx.dk/
 - kalibrate: https://github.com/xtrx-sdr/kalibrate-rtl
 - osmo-trx: https://github.com/xtrx-sdr/osmo-trx


# FAQ
## I don't see XTRX in `lspci` output
1. XTRX is installed into miniPCIe slot. Make sure that your system actually has PCIe lanes routed. Some system routes only USB lines, in the case XTRX LED near edge will be blinking ON-OFF-ON-OFF-.. pattern with ~3sec period. If this this the case install XTRX in other slots.
2. ~~XTRX is installed into PCIe slot via miniPCIe to PCIe adapter which is not our PCIeX2_FE card. This this rare but we found that some adapters are incompatible in some way (we're investigating the issuie) that prevents system to boot, try another adapter.~~
3. XTRX is blinking with ON-OFF-OFF-OFF-ON-OFF-OFF-OFF pattern. It means XTRX hasn't been enumerated on PCIe bus. Try to use other PCIe or miniPCIe slot if available. Please contact <xtrx@fairwaves.co> with details of you installation and system details for future assistance.
4. In all other cases fell free to contact us <xtrx@fairwaves.co>

## Which board/laptop can I use with XTRX in PCIe mode?
**You can use any board/laptop with XTRX with a USB3 adapter. Below asumes that you want to use XTRX in PCIe mode**

Generally, your board should have the miniPCIe socket **with the PCI lane**. Most laptops don't work, since generally they have PCI lane only on half-sized miniPCIe sockets (used for WiFi devices) while full-sized sockets designed mostly for use with SATA and USB lanes only. Industrial SBCs frequently have at least one such socket thus are more suitable for XTRX.

These devices were tested by our customers or us and were reported working fine with XTRX:
 - [UpSquared](https://up-board.org/upsquared/specifications/) by UpBoard
 - [Jetson TK1](https://www.nvidia.com/object/jetson-tk1-embedded-dev-kit.html) by NVidia
 - various devices by [LEX](http://www.lex.com.tw/), specifically [TERA + 2I610DW](http://www.lex.com.tw/products/TERA-2I610DW.html) and [3I610CW](http://www.lex.com.tw/products/3I610CW.html)
 - various SBCs by [PC Engines](https://www.pcengines.ch/), specifically [apu1d4](https://www.pcengines.ch/apu1d4.htm), [apu2d4](https://pcengines.ch/apu2d4.htm), and [apu4b4](https://www.pcengines.ch/apu4b4.htm) (may be suitable for 1 XTRX only, check the specs)
 - [Newport GW6300](http://www.gateworks.com/product/item/newport-gw6300-single-board-computer) by GATEWORKS
 - [ROCKPro64](https://www.pine64.org/rockpro64/) by Pine64
 - [IPC3](https://www.fit-pc.com/web/products/ipc3/) by fit-PC
 - [ARTiGO A1200](https://www.viatech.com/en/support/eol/artigo-a1200-eol/) by VIA (see #28)

These SBCs will probably work with XTRX but were not actually tested. Try on your own risk!
 - various SBCs by [AAEON](https://www.aaeon.com/en/c/embedded-single-board-computers/)
 - various SBCs by [Advantech devices](https://www.advantech.com/products/embedded-single-board-computers/sub_c427052f-f55f-4d47-9a84-3a11ed5295df)
 - various devices by [Logic Supply](https://www.logicsupply.com/)
 - [MPT-7000V](https://www.ibase.com.tw/english/ProductDetail/IntelligentTransportation/MPT-7000V) by IBASE; suitable for 1x XTRX only
 - [Nuvo-5100VTC](https://www.neousys-tech.com/en/product/application/in-vehicle-computing/nuvo-5100vtc); seems suitable for 2x XTRX (4x miniPCIe sockets, 2x of them supports USB signals only)
 - [ECS-9110](http://www.vecow.com/dispPageBox/vecow/VecowCT.aspx?ddsPageID=PRODUCTDTL_EN&dbid=4357925395) by Vecow most likely would work
 - [EC500-KH](https://www.dfi.com/Product/Index/1412) and [EC70B-SU](https://www.dfi.com/Product/Index/125) by DFI most likely would work
 - some older versions of Intel NUC which have miniPCIe may work
 - most likely Dell Latitide D420 would work; it has full-sized miniPCIe (used for WiFi by default) and no PCI whitelist in BIOS
 - some outdated EeePCs like 901 and 701 by asus may work (assuming based on the same premises as for Dell Latitude D420)
 - any device with full-featured PCIe x2 socket or USB3+ port if you use XTRX with PCIe x2 Front End or USB 3 Adapter respectively (see [CrowdSupply page](https://www.crowdsupply.com/fairwaves/xtrx) for details and order)
You can also check other SBCs by mentioned vendors.

## XTRX installed into miniPCIe slot prevents the system from booting (don't get even to BIOS)
This problem can happen if your motherboard provide SMB signals to miniPCIe connector. Most system doesn't. The XTRX rev.4 use reseved pins that are held in hi-Z. However, during the startup DC/DC chip isn't initialized and XTRX actually pulling down the SMB DATA line that mostly connected to critical to boot chips. We're working on firmware fix but currently hardware modification (cutting the fuse on the bottom side of xtrx near the edge) is the only solution.
Cut the fuse with a sharp knife. Make sure not to use much pressure since you can accidently cut other traces.

![smb_fuse](https://xtrx-sdr.github.io/images/smb_data_fix.jpg)

Systems that known to have such problem

| Vendor   | Model        | Comments |
| -------- | ------------ | -------- |
|  Jetway  | NP591        |   |
|  Via     | Artigo A1200 |   |
|  Intel   | DQ77KB       | SMBus even mentioned in the block diagrm [doc](https://www.intel.com/content/dam/support/us/en/documents/motherboards/desktop/dq77kb/dq77kb_techprodspec08.pdf) |
|  Lex     | 2I610HW      | Both miniPCIe have SMBus routed |


## I connected XTRX directly via microUSB cable and I don't see it `lsusb` 
Unfortunatly USB2 PHY software isn't ready and wasn't pushed in the master. You can check this https://github.com/xtrx-sdr/images/issues/9 for more deatils and suggestions.

## Is Windows driver support planned?
We plan to add Windows PCIe drivers but we don't have specific schedule for this right now. If it's blocking factor please contact us <xtrx@fairwaves.co>. However, all other software layers are windows compatible and USB3380 adapter should work via WinUSB.

## I've got a DMA buffer issue on my ARM device
You should probably increase DMA coherent pool using `coherent_pool=32M` kernel option in case you see following lines in the `dmesg` output:
```
[ 7.171608] xtrx: Failed to allocate 31 DMA buffer
[ 7.171632] xtrx 0000:01:00.0: Failed to register TX DMA buffers.
```
For the details see issue #37.
