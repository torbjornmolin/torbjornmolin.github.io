---
title: "IMOU Ranger RC Cameras"
date: 2025-03-28T18:53:01+01:00
draft: false
---

# Background

Recently I've bought a couple of IMOU Ranger RC wifi cameras in order to learn more about embedded Linux IOT devices. These cameras cannot out of the box be used stand alone, they can only be used with the IMOU App connected to IMOU cloud services. That is more or less useless to me, so my goal was to install custom firmware on them. The firmware I picked ended up being [OpenIPC](https://openipc.org/) due to hardware support.

There are a few versions of these cameras available, I have the 3- and 4-megapixels versions, but I've mainly focused on the 4 mp version.

# IMOU Ranger RC 3 MP

From the outside the cameras all look the same except for the color of the plastic, but on the inside they use slightly different hardware. The 3 MP uses a Sigmastar SSC 332 SOC which seems to be uncommon, but also seems to run fine with OpenIPC Sigmastar SSC 333 images. The wifi module in the one I have is based on iComm SV6115 and seems to work with the driver from OpenIPCs BR2_PACKAGE_SSV635X_OPENIPC-package.

# IMOU Ranger RC 4 MP

This camera is based on the Sigmastar SSC 337 SOC and an iComm SV6155P wifi module. The wifi module should work with the BR2_PACKAGE_SSV615X_OPENIPC-package from OpenIPC (you also need the mac80211 kernel module). I haven't gotten this to work, however. The driver finds the hardware but wifi connections fail in the handshake phase with "wrong key" as the error message.

## Ports

There is a UART header labeled J3 on the board next to the power input. Pin 3 is ground and the two pins before that are TX and RX (unsure of the order, it's usually quick trial and error each time I connect). The power connector has four unused pins, these are ethernet (GND, 5V, Ethernet green, green/white, orange, orange/white). The connectors themselves are 1,25mm pitch Molex Picoblade (or compatible). I found a box of pre-crimped wires and connector housings online as crimping wires with connectors that size seemed fiddly.

The wifi module get's power from GPIO pin 46, so that needs to be set high before the module will work. The SD card reader needs GPIO 45 to be pulled high, then low, before boot to work.

GPIOs:

| PIN | Function          |
| --- | ----------------- |
| 31  | Y-axis motor      |
| 36  | Y-axis motor      |
| 37  | Y-axis motor      |
| 38  | Y-axis motor      |
| 14  | X-axis motor      |
| 15  | X-axis motor      |
| 16  | X-axis motor      |
| 17  | X-axis motor      |
| 44  | IR LED            |
| 45  | SD Card reader    |
| 46  | USB (wifi module) |
| 61  | Green front LED   |
| 79  | Red front LED     |
| 76  | IR shutter open   |
| 77  | IR shutter close  |

## Flashing firmware

The stock firmware U-Boot can be accessed using the password '\*', keep sending it as soon as the device gets power. It is however a bit limited but if you connect ethernet using the pins on the power connector you might be able to use TFtp and flash new firmware that way.

Another way is to use [flashrom](https://www.flashrom.org/) with the mstarddc_spi programmer. You may have to build a custom version of flashrom, as not all distributed binaries include all programmer drivers. The i2c interface on the camera is accessed using the same pins as the UART, so once you have that working you just need to connect the same leads to your i2c interface (I used a Raspberry PI).

```
flashrom -p mstarddc_spi:dev=/dev/i2c-1:0x49 -v backup_image.bin
```

Will then get you a backup of the current content of the flash (provided that your i2c interface is available on /dev/i2c-1)
Once you have that, use flashrom to write new content to the flash (the chips on my cameras were 8mb NOR flash chips).
