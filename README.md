#Rajesh K Jeyapaul, jrkumar@in.ibm.com

Intel Edison and Bluemix Powered Air Traffic Ground station

Note: Most of the work around this done using RasperryPi. When I tried with Intel Edison,faced some challneges.Documenting
the same for a quick integration with Intel Edison and IBM Bluemix.

Step 1: Understand the Hardware Device

Device specification: NooElec - RB20T2 SDR & DVB-T (NESDR Mini 2)

Step 2: Connect the device to Intel

(a) COnnect to the power source as shown in the picture
(b) Connect to the Intel Edison device with the ip address assigned to it. Refer relevant documents on how to connect to Edison.

Step 3: Setup Edison environment for USB driver installation

"opkg" can be used to install the required filesets in Edison. The base feed for the same should be updated
(a) Update /etc/opkg/base-feeds.conf contents with below URL details:
src/gz all http://repo.opkg.net/edison/repo/all
src/gz edison http://repo.opkg.net/edison/repo/edison
src/gz core2-32 http://repo.opkg.net/edison/repo/core2-32
(b)opkg update
(c) opkg install libusb-1.0-dev

step 4: Build and Install RTL-SDR Driver @ Edison

cd ~
$ git clone git://git.osmocom.org/rtl-sdr.git
$ cd ~/rtl-sdr
$ cmake ./ -DINSTALL_UDEV_RULES=ON -DDETACH_KERNEL_DRIVER=ON
$ make
$ sudo make install
$ sudo ldconfig

Step 5: Build Dump1090 server , which is configured to receive the ADS-B raw message packets
cd ~
$ git clone https://github.com/MalcolmRobb/dump1090
$ cd ~/dump1090
$ export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
$ make
 
step 6: Run the Dump server (Ensure that the SDR device is connected with Edison and  Powered ON )
cd ~/dump1090
$ ./dump1090 --raw --net

At this stage, the SDR is tuned to 1090MHz frequency and listen for client TCP connections on port 30002.
All the connected clients will receive the raw ADS-B messages.

Debug:
Issue: Error opening the RTLSDR device: Device or resource busy
Soln : Reboot Edison

output sample: a80007000000000000000081225d
How to interpret this data - Refer Decoding guide - http://adsb-decode-guide.readthedocs.io/en/latest/content/introduction.html
step 7: Decode the Raw message to readable
(a) clone the git , onto the Edison
(b) Modify the host to the IP address of Edison (line 5)
var host = '192.168.1.144';
var port = 30002;
(c)Run it as : Run the nodejs - node node-examplenet.js

output sample : 
Flight status : 05 (air or ground), 0 - airborne
a80007000000000000000081225d - Flight Status 5: Special Position Identification. Airborne or Ground

Credits: https://github.com/grantmd/node-adsb, 
https://github.com/ibm/air-traffic-control
