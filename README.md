# Main XTRX images & firmwares

This is preview of r3 only XTRX support software. Any problem running prebuild or compiled libraries from this revision report directly to github. This preview is for Linux only

## Structure
 - `binaries` - contains prebuild host packages. Currently host libraries are provided only for Ubuntu 16.04 x86_64 target.
 - `sources` - source code referrence of what were used. It has submodules of packages. See *Manual Compilation* for more details
 - `firmwares` - FPGA images for reflashing. (not yet published)



## Manual compilation of host libraries from `source` directiry

You need C and C++ compiler to build host libraries. `libusb1` is also required if you're using USB3 adapter. For debian-based system:
```
% sudo apt-get install build-essential libusb-dev cmake dkms
``` 
First you need to clone relevant revisions of source modules:
```
% git submodule init
% git submodule update
```
After that you need to build libraries:
```
% mkdir -p build
% cd build
% cmake ..
% make
```
When everything is built you can install it:
```
% sudo make install
```
If you're using PCIe mode you also need to install kernel driver. 
```
% sudo /usr/sbin/dkms add -m xtrx -v"0.0.1-1"
% sudo /usr/sbin/dkms build -m xtrx -v"0.0.1-1"
% sudo /usr/sbin/dkms install -m xtrx -v"0.0.1-1"
```

If everything is Ok, you can load the driver:
```
modprobe xtrx
```
and it should appears in `/proc/modules`


# Getting started
## Working with PCIe
PCIe device is `10ee:7012`. If driver `xtrx` is loaded than character device `/dev/xtrx0` should appear.


## Working with USB3 adpapter
USB3 adapters appears under `0525:3380`. You can check it by running `lsusb`:
```
%lsusb
Bus 002 Device 002: ID 0525:3380 Netchip Technology, Inc. 
```

## Checking if it works
You can check that device is working:
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

If it pases you can go to writting your application :) or installing one of already supported:

 - GNU Radio: https://github.com/xtrx-sdr/gr-osmosdr
 - sdrangel: https://github.com/xtrx-sdr/sdrangel
 - gqrx (no special branch, use official): http://gqrx.dk/
 - kalibrate: https://github.com/xtrx-sdr/kalibrate-rtl









