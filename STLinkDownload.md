# Introduction #


This program is a download and interaction utility for several versions of the STMicro USB STLink programming/debug interface.  The STLink is found on STM8 and STM32 "Discovery" development boards, as well as in a stand-alone module.

This utility was originally written December 2010 by Donald Becker and William Carlson for the STM32VLDiscovery.  It was later expanded to support the "STLink v2" found on the later development boards, with help from Hugo Becker.


This program may be used under the terms of the Gnu General Public License,(GPL) v2 or v3.  Distribution under other terms requires an explicit license from the authors.

The v1 communication is based on an emulated standard USB mass storage device, with its SCSI command protocol. The STLink operations are encapsulated in vendor-specific SCSI commands.  The emulation is crudely done, with only enough implemented to fool certain versions of Windows.  This causes operational problems with Linux (which takes many seconds to decide that multiple retries still produces the same broken responses), and is an implementation blocker on Apple (which treats SCSI encapsulated communication as a security issue).

The STLink v2 does away with the SCSI encapsulation.  It uses the same protocol on raw USB endpoints.

For the v1 we directly use the SCSI Generic (sg) ioctl() and data structures to communicate with the STLink.  The alternative of using the sgutils2/pass-through libraries might have taken less initial development time, but otherwise add no value.  Those libraries are not always installed, causing extra build and installation headaches. Most of their functions are simple wrappers that add more complexity
than they hide.  Their only significant benefit, combining status values from a unpredictable mix of different driver, host adapter, transport and target combinations, does not apply when those are all emulated by the STLink firmware.

For the v2 interface we relax our "no external library dependency" requirement slightly to use libusb-1.0.  This decision may be revisited, as libusb-1.0 still requires enhanced file system permissions.  (And besides, a library that explicitly encodes the version number in the library base name can't be trusted with long-term stability.)


# References #

ST Micro Applications Notes
> AN3154 Specification of the CAN bootloader protocol
> AN3155 Specification of the USART bootloader protocol
> AN3156 Specification of the USB DFU (Direct Firmware Upload) protocol
Related documents
> http://www.usb.org/developers/devclass_docs/DFU_1.1.pdf
> > [Note: I have omitted all DFU support from this release. ](.md)

> USB Device Class Definition for Mass Storage Devices:
> > www.usb.org/developers/devclass\_docs/usbmassbulk\_10.pdf
Private conversation with Jon Masters regarding ubiquity of libusb-1.0 on Linux platforms.

# Build notes #

The code was written to be as generic as possible.  It requires no special compile options or libraries.  (Exception noted above.)


> gcc -o stlink-download.o stlink-download.c

# Usage notes #

This is a deeply broken device.  They apparently built the mass storage
interface by cribbing the USB device table from a 32MB flash stick.  But
the device doesn't have 32MB, only hard-wired responses that presents
three tiny 'files'.  Not arbitrary responses, only ones to the exact
USB commands that Windows would use, and only accurately filling in the
fields that Windows uses.

The result is that non-Windows machines choke on this device.  (It's likely
that future Windows versions will as well.)  Linux automatically checks the
partition table and validates the extents.  It takes several minutes of
periodic retries before the kernel gives up and decides it is truly broken.

Distributed with this utility is a udev rule file that speeds up
the process by avoiding an automatic mount.

The "Capt'ns Missing Link" people also suggest doing
> modprobe -r usb-storage && modprobe usb-storage quirks=483:3744:l
or adding the equivalent to /etc/modprobe.conf or /etc/modprobe.d/local.conf
> options usb-storage quirks=483:3744:l