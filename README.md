sierra_wireless_forward_port
============================

I recently purchased a Sierra Wireless (now Netgear) [Aircard
320U](http://www.netgear.com/service-provider/products/mobile-broadband/usb-modems/aircard_320U.aspx)
USB 3G/4G modem for some testing.  I selected the Aircard 320U based
partly on its technical specs (compatibility with the major New Zealand
providers), as well as the fact they actually bother to provide support
for Linux.  

The unit is supported by two kernel modules.  _sierra_ is a
USB-serial module, which enumerates a number of `/dev/ttyUSB*` serial
devices.  These allow direct communication with the modem.  For example,
I'm using `/dev/ttyUSB2` to send AT-style commands to the modem.

The *sierra_net* module appears as a USB-network driver.  As such,
the modem appears directly as a network device (`wwan0` for me) without any additional
mucking about with PPP, etc.

Sierra provides
[instruction](http://mycusthelpadmin.net/SIERRAWIRELESS/_cs/AnswerDetail.aspx?sSessionID=&aid=44)
for building their kernel source.  Unfortunately, the last entry is for
"v3.2_1740" for kernel 3.0.x.    As I'm using a relatively recent Ubuntu
installation, I'm running a 3.8 kernel.

The Linux 3.8 kernel provides the *sierra* (no version number) and
*sierra_net* (v2.0) drivers, but I was unable to get these drivers to
enumerate the Aircard 320U.  The tarball from Sierra lists *sierra*
v.1.7.40 and *sierra_net* v.3.2, but out of the box
the Sierra code would not build under kernel 3.8.x (changing USB APIs,
I imagine).

It seemed like a case of the kernel containing old code which was
being ported forward with each new kernel rev, while the vendor was providing
updated code, but not keeping up with kernel dev.  Seemed like a straightforward call
to forward-port the driver, which you see here.

I should disclaim I didn't do this using my deep knowledge of the Linux
kernel APIs, but by a "monkey-see-monkey-do" port between the Sierra
code and the current kernel code.  It works for me, on Ubuntu 13.04,
with an Aircard 320U.  Maybe it will work for you too?

I was briefly testing under Ubuntu 12.10, so I also ported to 3.6. I've also been able to use this code to run the Sierra on a RaspberryPi running Raspbian (kernel 3.6.y).  Actually building the kernel modules was mildly inconvenient, instructions can be found on the internet.

Instructions
------------

You'll need to have the appropriate build tools and kernel headers installed, of course.

```
cd 3.8
make
sudo make install
```

**Please note:**  `make install` will copy the kernel drivers to `/lib/modules/...` and overwrite the existing kernel drivers!

Then plug in the modem.  I get:

```
[ 9943.168294] usb 1-4.4.4: new high-speed USB device number 19 using ehci-pci
[ 9943.261665] usb 1-4.4.4: config 1 has an invalid interface number: 9 but max is 5
[ 9943.261669] usb 1-4.4.4: config 1 has an invalid interface number: 7 but max is 5
[ 9943.261672] usb 1-4.4.4: config 1 has no interface number 2
[ 9943.261674] usb 1-4.4.4: config 1 has no interface number 5
[ 9943.263162] usb 1-4.4.4: New USB device found, idVendor=0f3d, idProduct=68aa
[ 9943.263165] usb 1-4.4.4: New USB device strings: Mfr=3, Product=2, SerialNumber=4
[ 9943.263168] usb 1-4.4.4: Product: AirCard 320U
[ 9943.263170] usb 1-4.4.4: Manufacturer: Sierra Wireless, Incorporated
[ 9943.263173] usb 1-4.4.4: SerialNumber: 1234567890123456
[ 9943.264955] sierra 1-4.4.4:1.0: Sierra USB modem converter detected
[ 9943.267164] usb 1-4.4.4: APM supported, enabling autosuspend.
[ 9943.267241] usb 1-4.4.4: Sierra USB modem converter now attached to ttyUSB0
[ 9943.267309] sierra 1-4.4.4:1.1: Sierra USB modem converter detected
[ 9943.268037] usb 1-4.4.4: APM supported, enabling autosuspend.
[ 9943.268104] usb 1-4.4.4: Sierra USB modem converter now attached to ttyUSB1
[ 9943.268167] sierra 1-4.4.4:1.3: Sierra USB modem converter detected
[ 9943.268790] usb 1-4.4.4: APM supported, enabling autosuspend.
[ 9943.268862] usb 1-4.4.4: Sierra USB modem converter now attached to ttyUSB2
[ 9943.268936] sierra 1-4.4.4:1.4: Sierra USB modem converter detected
[ 9943.269663] usb 1-4.4.4: APM supported, enabling autosuspend.
[ 9943.269728] usb 1-4.4.4: Sierra USB modem converter now attached to ttyUSB3
[ 9943.269883] scsi19 : usb-storage 1-4.4.4:1.9
[ 9943.271342] sierra_net 1-4.4.4:1.7 wwan0: register 'sierra_net' at usb-0000:00:1a.7-4.4.4, Sierra Wireless USB-to-WWAN Modem, 22:c6:d7:22:06:07
[ 9944.269553] scsi 19:0:0:0: Direct-Access     SWI      SD Card          2.31 PQ: 0 ANSI: 2
[ 9944.269937] sd 19:0:0:0: Attached scsi generic sg2 type 0
[ 9944.272435] sd 19:0:0:0: [sdb] Attached SCSI removable disk
```

which shows four serial devices (no, I don't actually know what they all do), the network device as `wwan0` and the block device driver for the MMC card slot.  Neat!  Yes, there are even non-fatal warnings!

On the application side, I've found `modem-manager` supports this card well, and actually scraped its AT commands when writing my own test scripts.  I haven't tried `wvdial` or etc, but I suppose they could be made to work.
