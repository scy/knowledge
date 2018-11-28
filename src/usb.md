# Universal Serial Bus (USB)

_It was supposed to save us all, now it's part of the problem. ([I've ranted about this.](https://twitter.com/scy/status/987642608649490432))_

## Per-Port Power Switching (PPPS)

Also known as _Hub Port Power Control_ (in the USB 2.0 spec), this is a feature of the USB protocol that allows a host to ask a hub to power down some or all of its device-facing ports.
Surprisingly few people know about this, which might be related to surprisingly few hubs actually implementing this correctly.

In Linux, using `lsusb -v` and looking for `Per-port power switching`, you can find out if your hub supports it.
Note that this doesn't just mean _external_ hubs; the USB ports built in to your machine are a "hub" on their own.

You can use a tool like [uhubctl](https://github.com/mvp/uhubctl) to control the power state of your hub(s), but usually, that's where the problems start:

* Most hubs claim to support this, but only disconnect the data lines, leaving 5V power to the device untouched.
  This is usually because the hub manufacturer skipped the necessary switches/transistors on their board and the power lines are hard-wired to 5V.
  There are people who [modified their USB hub hardware to fix this](https://befinitiv.wordpress.com/2014/02/02/hacking-per-port-power-switching-to-an-usb-hub-2/), but it requires electronics and soldering skills.
* Backward-compatible USB 3 hubs contain both a USB 3 and a USB 2 controller; _both_ have to be told to switch off the port.
  uhubctl should do this automatically though.
* If your hub has more than 4 ports, chances are that it actually consists of multiple hubs daisy-chained to each other.
  A seven-port hub would for example contain two four-port controllers, with the second controller connected to port 4 of the first controller.
  Disabling one of the ports another controller is connected to can lead to undefined behavior, as the downstream controller(s) might not be re-initialized correctly when you re-enable them.

If you're still determined you want to try this adventure, the [uhubctl GitHub page](https://github.com/mvp/uhubctl) contains a list of devices that are known to be compatible.
Finding a place to buy them is a quest on its own though.

## USB 3 hubs on a Raspberry Pi

RasPis have a bad reputation when it comes to their USB connectivity.
Using a USB 3 hub on a Pi that only supports USB 2 is one of the reasons for that.
_(Note: As of November 2018, when I wrote this, this problem applied to all Raspberry Pi models, including the 3B+.)_

According to [raspberrypi/firmware#64](https://github.com/raspberrypi/firmware/issues/64), using USB 1 devices on USB 3 hubs can sometimes cause problems.
One of the symptoms is repeated `device descriptor read/64, error -71` messages in `dmesg` and the device not appearing.
If I understand the thread correctly, it's caused by faulty firmware in the hubs, with the Pi's firmware in turn not being tolerant enough to compensate.

Funny enough, I seem to have had this problem only with USB **2** devices on a USB 3 hub, so my suggestion is to avoid using USB 3 hubs on a Pi altogether.

The official workaround by the way is to connect a USB 2 hub to one of the ports of the USB 3 hub and to connect your USB 1 devices to that one instead.
I have not tried this method.
