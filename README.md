# XTRX FPGA images & pre-compiled binaries

This repository hosts FPGA images and pre-compiled binaries for XTRX. Only rev3 hardware and only Ubuntu 16.04 x86_64 binaries are published at this moment.

Please report any problems running pre-compiled libraries from this repository directly this repo's [issues](https://github.com/xtrx-sdr/images/issues).

## Structure
 - `binaries` - pre-compiled host binary packages.
 - `sources` - reference source code used for building the binaries. Uses git submodules to reference to the relevant git repositories. See *Manual Compilation* for more details.
 - `firmware` - FPGA images for reflashing. (not yet published)



## Manual compilation of host libraries from the `sources` directory

You need C and C++ compiler to build host libraries. `libusb1` is also required if you're using a USB3 adapter.
 - For Debian-based systems:
```
% sudo apt-get install build-essential libusb-dev cmake dkms
```
 - For Fedora systems:
```
sudo dnf groupinstall 'Development Tools' 'C Development Tools and Libraries' dkms libusb-devel cmake
```

First, you need to clone relevant revisions of source modules:
```
% git submodule init
% git submodule update
```
Then you can build the libraries by running the following from the `sources` directory:
```
% mkdir -p build
% cd build
% cmake ..
% make
```
Please not that by default the SoapySDR is enabled by default. If you don't use it, then you can disable it by adding `-DENABLE_SOAPY=OFF` to `cmake`.

When everything is built you can install them:
```
% sudo make install
```
If you're connecting XTRX over a (mini)PCIe bus, you also need to install a kernel driver. 
```
% sudo /usr/sbin/dkms add -m xtrx -v"0.0.1-1"
% sudo /usr/sbin/dkms build -m xtrx -v"0.0.1-1"
% sudo /usr/sbin/dkms install -m xtrx -v"0.0.1-1"
```

If everything went well, you can now load the driver:
```
modprobe xtrx
```
and it should appear in `/proc/modules`


# Getting started
## Working with XTRX over PCIe bus
PCIe device ID is `10ee:7012`. If driver `xtrx` is loaded correctly, a character device `/dev/xtrx0` should appear.


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
 - gqrx (no special branch, use official): http://gqrx.dk/
 - kalibrate: https://github.com/xtrx-sdr/kalibrate-rtl

