# XTRX FPGA images & pre-compiled binaries

This repository hosts FPGA images and pre-compiled binaries for XTRX. Only rev4 hardware and only Ubuntu 16.04 x86_64 binaries are published at this moment.

NOTE: rev3 hardware support is phasing out, please contact <xtrx@fairwaves.co> in order to get support.

Please report any problems running pre-compiled libraries from this repository directly this repo's [issues](https://github.com/xtrx-sdr/images/issues).

## Structure
 - `binaries` - pre-compiled host binary packages.
 - `sources` - reference source code used for building the binaries. Uses git submodules to reference to the relevant git repositories. See *Manual Compilation* for more details.
 - `firmware` - FPGA images for reflashing. (not yet published)



## Manual compilation of host libraries from the `sources` directory

You need C and C++ compiler to build host libraries. `libusb1` is also required if you're using a USB3 adapter. For Debian-based systems:
```
% sudo apt-get install build-essential libusb-1.0-0-dev cmake dkms python-cheetah
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
% sudo ln -s /usr/local/src/xtrx-0.0.1-1/ /usr/src/
```

And now instruct DKMS to build the module:
```
% sudo /usr/sbin/dkms add -m xtrx -v "0.0.1-1"
% sudo /usr/sbin/dkms build -m xtrx -v "0.0.1-1"
% sudo /usr/sbin/dkms install -m xtrx -v "0.0.1-1"
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

