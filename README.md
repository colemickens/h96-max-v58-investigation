# h96-max-v58 investigation

<!--toc:start-->
- [h96-max-v58 investigation](#h96-max-v58-investigation)
  - [dumps](#dumps)
  - [uart access](#uart-access)
  - [usb otg access](#usb-otg-access)
  - [adb access](#adb-access)
  - [-> recovery](#recovery)
  - [-> fastbootd](#fastbootd)
  - [-> bootloader](#bootloader)
  - [-> rockusb mode](#rockusb-mode)
  - [backup/dump device](#backupdump-device)
  - [adb pull](#adb-pull)
  - [unknown??](#unknown)
<!--toc:end-->

<img src="https://ae01.alicdn.com/kf/Sa5a4231fbb254cb0b8cf3cb479dd5655K.jpg" />

Rock Pi 5 should be shipping soon, but this thing is cute, in an enclosure, etc.
Maybe the Rock Pi 5 will help support this thing too.

Hopefully here we can collect info about the H96 Max V58 device
 - dump the Android boot part
 - get the DTB?
 - figure out if they're using a kernel fork and if we can get access etc


Item Purchased:
- https://www.aliexpress.com/item/3256802713801854.html

## dumps

```
❯ adb shell
rk3588_box:/ # ls /dev/block/by-name
backup  baseparameter  boot  cache  dtbo  logo  metadata  misc
mmcblk0  mmcblk0boot0  mmcblk0boot1
recovery  security  super  trust  uboot  userdata  vbmeta

❯ cat /proc/device-tree/compatible; echo
rockchip,rk3588-nvr-demo-v10-androidrockchip,rk3588

❯ file boot.img
boot.img: Android bootimg, kernel (0x10008000), ramdisk (0x11000000), second stage (0x10f00000), page size: 2048, cmdline (console=ttyFIQ0 firmware_class.path=/vendor/etc/firmware init=/init rootwait ro loop.max_part=7 androidboot.console=ttyFIQ0 and)
```

- note the weird console=tty val

- internet archive:
  - details page: https://archive.org/details/h96-max-v58-dump
  - direct dl listing: https://archive.org/download/h96-max-v58-dump

## uart access
- baud rate is 1500000
  - use the hw597 uart thing (link aliexpress)
  
## usb otg access
- in certain scenarios, the device will be accessible over USB (think OTG)
- use a USB-C to USB-A cable
- the USB-A end goes in the USB2 port on the device (black inner plastic bit)
- the USB-C end goes in your computer/adapter/etc
 
## adb access
1. Device settings
2. Tap "Android TV Build Version" x 6
3. Now you have dev menu.
4. Enable dev options, "over internet"
5. `adb connect <dev-ip>` from your PC

## -> recovery
1. Unplug device + ensure the device isn't usb connected to a PC
2. Hold reset.
3. Plug device.
4. Let go after... 3 seconds.
(If you hold too long, you will enter bootloader)

*notes:*
- "Sideload" doesn't seem to work...

## -> fastbootd
1. Enter recovery (see above)
2. Press "Enter Fastboot". (use usb-keyboard, or you can use IR remote)

*notes:*
- doesn't seem to do anything, doesn't show as fastboot device, incorrectly says
  secureboot or whatever is on

**unclear if this is useful, re-plugging USB-OTG here doesn't do anything**

## -> bootloader
1. Enter recovery (see above)
2. Press "Enter Bootloader". (use usb-keyboard, or you can use IR remote)
2. Connect to the board OTG style. (see above)

```
❯ fastboot devices
HCY58uk5hm8Tp85DwGAj	Android Fastboot
```

## -> rockusb mode
1. Unplug the board.
2. Connect to the board OTG style. (see above)
3. Hold reset button, plug the board in.
4. Release reset button after 6 seconds.

  relevant boot uart log:
  ```
  boot mode: recovery (misc)
  boot mode: None
  Android 12.0, Build 2022.6, v2
  Found DTB in boot part
  DTB: rk-kernel.dtb
  HASH(c): OK
  ANDROID: fdt overlay OK
  [snip]
  Model: Rockchip RK3588 NVR DEMO LP4 V10 Android Board
  download key pressed... entering download mode...
  RKUSB: LUN 0, dev 0, hwpart 0, sector 0x0, count 0x747c000
  \%
  ```
  (the last bit is animated, seemingly waiting for rockusb commands from PC)

  dmesg output:
   ```
  [90320.423807] usb 3-1: New USB device found, idVendor=2207, idProduct=350b, bcdDevice= 2.23
  [90320.423812] usb 3-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
  [90320.423815] usb 3-1: Product: USB download gadget
  [90320.423816] usb 3-1: Manufacturer: Rockchip
  [90320.423817] usb 3-1: SerialNumber: HCY58uk5hm8Tp85DwGAj
   ```

## backup/dump device

```
~
❯ rkdeveloptool ld
DevNo=1	Vid=0x2207,Pid=0x350b,LocationID=301	Loader
```


## adb pull

```

~
❯ adb root
restarting adbd as root

~
❯ adb pull /dev/block/by-name/boot /tmp/boot.img
/dev/block/by-name/boot: 1 file pulled, 0 skipped. 94.3 MB/s (50331648 bytes in 0.509s)

```

## unknown??

- package rkdumper ??
- "rkandroidtool" ??
