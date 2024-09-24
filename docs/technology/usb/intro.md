# Universal Serial Bus and Software Defined Radios

USB was invented way back in 1996 at 1.5Mbps, soon increased to 12Mbps. USB2.0 introduced in 2000 is of the most relevance to us, because the dominant Software Defined Radio chipset - the RTL-2832 and family - only supports USB2.0 and only supports 480Mbps operation.

USB3.0 introduced in November 2008 has a dirty little secret. The 5Gbps wires added by USB3 never ever carry USB2 data. A collection of RTL-2832 dongles plugged into a USB4, USB3.2, USB3.1, USB3 hub will all share the 480Mbps of the USB2 wires and the 20GT/s, 10GT/s, 5GT/s wires sit completely idle!

For reasons we believe the USB2 interface is limited to about 10-12MS/s (Million Samples per second, roughly analogous to 2MHz of bandwidth) for IQ-type things that online radio-loving humans like doing. So a single 10MHz dongle such as an Airspy will occupy most all of a USB2.0 Bus. And 5-6 RTL-SDR 2MHz dongles will equally occupy most all of a USB2.0 Bus. So the poor aircraft radio enthusiast is sitting here with all these USB ports and no bandwidth to use with them.

## Controllers

What matters is how many *controllers* the system has. Older computers with few or no USB3 ports often had two or three controllers, to share that 480Mbps love around. Newer computers with USB4 / Thunderbolt ports may have a single controller. Yes, it does 20GT/s USB4. But it only does 480Mbps USB2!

### How many controllers do I have?

Linux uses `lsusb` to interact with USB topologies from the console. The first layer of details are available to unpriviledged (plebian) accounts - such as `lsusb -t`, but full details require 'sudo' - `sudo lsusb -v | less`

For example, here is a random Pi4 I had laying about while writing this markdown...

<details open>
  <summary>Pi 4 lsusb -t = One USB2 Controller</summary>

```
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 5000M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 2: Dev 3, If 0, Class=Hub, Driver=hub/4p, 480M
            |__ Port 1: Dev 4, If 0, Class=Vendor Specific Class, Driver=usbfs, 480M
            |__ Port 1: Dev 4, If 1, Class=Vendor Specific Class, Driver=, 480M
            |__ Port 2: Dev 9, If 0, Class=Vendor Specific Class, Driver=usbfs, 480M
            |__ Port 3: Dev 6, If 0, Class=Vendor Specific Class, Driver=usbfs, 480M
            |__ Port 3: Dev 6, If 1, Class=Vendor Specific Class, Driver=, 480M
            |__ Port 4: Dev 7, If 0, Class=Vendor Specific Class, Driver=usbfs, 480M
            |__ Port 4: Dev 7, If 1, Class=Vendor Specific Class, Driver=, 480M
```

Of interest is the two controllers indicated with a leading `/`; the first is USB3 at 5Gbps and the second is USB2 at 480Mbps. All four USB devices on the hub (above named Port 1 Dev4, Port2 Dev9, Port3 Dev6, Port4 Dev7) are sharing that 480Mbps.

</details>

(Since this is a Pi4 I had laying about fair odds those are all SDR dongles...)

<details>
  <summary>Haswell i3-4310 lsusb -t = Three USB2 Controllers</summary>

```
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/6p, 5000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/14p, 480M
    |__ Port 11: Dev 2, If 0, Class=Wireless, Driver=btusb, 12M
    |__ Port 11: Dev 2, If 1, Class=Wireless, Driver=btusb, 12M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/8p, 480M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
```

Here we see a single USB3 controller with '6p', a USB2 controller with '14p' (running the wireless dongle), and two further USB2 controllers, each with '2p' and 480Mbps *each*
</details>

And now I would like to introduce a second tool named `lspci`, part of `pciutils` package on most distros and often not installed by default, this one generally requires 'sudo' and reports the PCI devices you have. Video Cards, Ethernet Card, Hard Drive Controllers, and of course USB Controllers are all typically PCI devices. So we can use `sudo lspci` to see what the USB controllers report themselves as - typically of interest is who made them.

<details>
  <summary>Haswell i3-4310 lspci</summary>

```
00:00.0 Host bridge: Intel Corporation 4th Gen Core Processor DRAM Controller (rev 06)
00:02.0 VGA compatible controller: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor Integrated Graphics Controller (rev 06)
00:03.0 Audio device: Intel Corporation Xeon E3-1200 v3/4th Gen Core Processor HD Audio Controller (rev 06)
00:14.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB xHCI (rev 04)
00:16.0 Communication controller: Intel Corporation 8 Series/C220 Series Chipset Family MEI Controller #1 (rev 04)
00:16.3 Serial controller: Intel Corporation 8 Series/C220 Series Chipset Family KT Controller (rev 04)
00:19.0 Ethernet controller: Intel Corporation Ethernet Connection I217-V (rev 04)
00:1a.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #2 (rev 04)
00:1c.0 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #1 (rev d4)
00:1c.3 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #4 (rev d4)
00:1c.4 PCI bridge: Intel Corporation 8 Series/C220 Series Chipset Family PCI Express Root Port #5 (rev d4)
00:1d.0 USB controller: Intel Corporation 8 Series/C220 Series Chipset Family USB EHCI #1 (rev 04)
00:1f.0 ISA bridge: Intel Corporation H87 Express LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 8 Series/C220 Series Chipset Family 6-port SATA Controller 1 [AHCI mode] (rev 04)
00:1f.3 SMBus: Intel Corporation 8 Series/C220 Series Chipset Family SMBus Controller (rev 04)
02:00.0 Ethernet controller: Qualcomm Atheros AR8161 Gigabit Ethernet (rev 10)
03:00.0 Network controller: Intel Corporation Centrino Wireless-N 2230 (rev c4)
```

I spy with my little eye, a USB Controller at 00:14.0, another at 00:1a.0, and a third at 00:1f.0. Those addresses together with the "C220 Series Chipset" moniker lead me to expect these controllers are in the South-Bridge, not on the CPU-die / 'uncore'. As befits an older architecture such as a Haswell.
</details>

Back to lsusb, let us look at a system that is a little more modern

<details>
  <summary>Comet-Lake i3-10110u = One USB2 controller</summary>

```
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/6p, 10000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/12p, 480M
    |__ Port 1: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
    |__ Port 2: Dev 3, If 0, Class=Vendor Specific Class, Driver=usbfs, 480M
    |__ Port 3: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
    |__ Port 3: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
```

So we have a 10GT/s USB4 or Thunderbolt controller there that is completely useless for radio games, and a single 480Mbps USB2 controller. This system has much more CPU power than the Haswell, but has no way to get radio data into the CPU. It also has no free PCI-E slots for upgrading.

</details>

### Can I get more controllers?

Plugging in USB Hubs will not help. They all share the USB2 wires, so you never get more bandwidth. Outside of a VIA Labs chip - VL670 / VL671, rumoured to be in use on some VR goggles, there is no way to have USB2 devices occupy the USB3 bandwidth of a system; and even those VL670 chips are technically out-of-spec.

So if you want more USB2 bandwidth, you need an additional Controller. USB plug-in-cards are not as common as they used to be, but are still availble. Sadly, most have a single controller, even if they have 1, 2, 4, or 7 USB ports.

Except for the StarTech PEXUSB3S44V, which has a PCI-E bridge chip and *four* USB Controllers on a PCI-Express gen2x4 add-in card.

I want to try an experiment. As it stands, this machine has two USB controllers, and `lspci | grep USB` reports
```
03:00.0 USB controller: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset USB 3.1 XHCI Controller (rev 01)
28:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Matisse USB 3.0 Host Controller
```

(One from the north-bridge and one from the south-bridge by the looks of it)

<details>
  <summary>Matisse Ryzen-5 3600 = Two USB2 controllers</summary>

```
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 10000M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 480M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 2: Dev 3, If 0, Class=Vendor Specific Class, Driver=rtl8192eu, 480M
        |__ Port 4: Dev 4, If 0, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 1, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 2, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 3, Class=Human Interface Device, Driver=usbhid, 12M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 10000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/10p, 480M
    |__ Port 8: Dev 2, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 8: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 9: Dev 3, If 0, Class=Hub, Driver=hub/3p, 12M
        |__ Port 1: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
        |__ Port 1: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
```

</details>

What happens if I was to plug a PCI-Express device such as the StarTech PEXUSB3S44V into this system? (Okay, being honest, I wanted to use the Haswell from before, but the card doesn't fit in the case!)

```
03:00.0 USB controller: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset USB 3.1 XHCI Controller (rev 01)
27:00.0 USB controller: Renesas Technology Corp. uPD720202 USB 3.0 Host Controller (rev 02)
28:00.0 USB controller: Renesas Technology Corp. uPD720202 USB 3.0 Host Controller (rev 02)
29:00.0 USB controller: Renesas Technology Corp. uPD720202 USB 3.0 Host Controller (rev 02)
2a:00.0 USB controller: Renesas Technology Corp. uPD720202 USB 3.0 Host Controller (rev 02)
2d:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Matisse USB 3.0 Host Controller
```

Here we see four new Renesas Technology USB controllers have appeared. Most Linux Kernels since like 2016 support this chip, so out-of-the-box experience is good.

<details>
  <summary>Matisse Ryzen-5 3600 + StarTech PEXUSB34S44V = Six USB2 controllers</summary>

```
/:  Bus 12.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 10000M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
/:  Bus 11.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 480M
    |__ Port 2: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        |__ Port 2: Dev 3, If 0, Class=Vendor Specific Class, Driver=rtl8192eu, 480M
        |__ Port 4: Dev 4, If 0, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 1, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 2, Class=Audio, Driver=snd-usb-audio, 12M
        |__ Port 4: Dev 4, If 3, Class=Human Interface Device, Driver=usbhid, 12M
/:  Bus 10.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 5000M
/:  Bus 09.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
/:  Bus 08.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 5000M
/:  Bus 07.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
/:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 5000M
/:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 5000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/1p, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 10000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/10p, 480M
    |__ Port 8: Dev 2, If 1, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 8: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
    |__ Port 9: Dev 3, If 0, Class=Hub, Driver=hub/3p, 12M
        |__ Port 1: Dev 4, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
        |__ Port 1: Dev 4, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
```

</details>

So this machine could now, in theory have 30 to 36 of 2MS/s dongles attached and with 6 CPU cores and 16GB of RAM, it might even keep up with them. Maybe.

(Off on another tangent, gen2x4 is 40GT/s on paper, 23Gbps observed. Four USB3 ports at 5GT/s on paper would crowd that slot and the system might not sustain full performance if I was to plug a bunch of USB3 HDD in. But since I only care about the USB2 bandwidth, &lt;2GT/s aggregate of USB2 fits on even a gen2x1 slot. So if your motherboard only has x1 slots and you can physically get the x4 card into one, the bandwidth is Good Enough)

## Bandwidth Measurements

There is a way to estimate current utilisation on a Linux system using ... &lt;details go here&gt;

Or you can use `rtl-test` on an unallocated RTL SDR dongle. Plug in multiple dongles to one hub, check with `lspci -t` they are all on a single controller, and then spark up `rtl-test` on each in turn until it starts reporting sample drops. At that point, the USB controller (or the CPU) is no longer "keeping up". (You can check cpu utilisation with `top`)

## Bandwidth calculations

### The Protocol

USB2 uses 8b10b encoding at 480MHz, so only 4/5 of the paper number is available (384Mbps). The USB framing protocol has 13-19 bytes of header on each "packet" (need to know how big packets are to know if that matters. 1 byte payload + 13 header is terrible. 1024 byte payload + 19 byte header not so much). Within the protocol, roughly 11% of packets are used for management and status, so it is not unreasonable to decide a "480Mbps" USB2 link only has 240Mbps (30MByte/s) of actual usable bandwidth for payload - such as HDD contents or SDR data

### The Radio Data

"Google says I/Q at 2.4MS/s is 38.4Mbps of data" -- Sitting.duc, 2023-09-12

"Where the heck did I find that number?!" -- Sitting.duc, 2024-09-24