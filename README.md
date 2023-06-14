# Enable S3 Sleep on Rog G14/15 2021/2022 using OpenCore
This tutorial helps you enable the hidden S3 sleeping mode on your laptop without modifying your BIOS. (Safest way, using OpenCore Bootloader)

Support tested : Asus Rog G14 2022 GA402RJ BIOS V312/318

This should be working for other models as well as the 2021/2023 models (and more, even more brands, you can test out, if not working you can simply chose the original boot option and delet the file I provided), you can test and share the results in dissucssion.

I've seen many disscussions on how to enable S3 sleep on the laptop. Some using RU to modify the BIOS, but it might cause a damage to the BIOS and that might be hard to recover for common users, also RU was [reportedly not working on the 2022 model](https://gist.github.com/raenye/d6645d7039a6136ccfb055e0f8517698#important=update). There are two more helpful works I read about, one using [Colver (another hakintosh bootloader)](https://gist.github.com/raenye/d6645d7039a6136ccfb055e0f8517698) and [another](https://gitlab.com/marcaux/g14-2021-s3-dsdt) confirming modifying DSDT.aml could fix sleeping issues under linux (for 2021/2022 models and ROG G15 as well).

- [Introduction](#introduction)
- [Why Using OpenCore](#why-using-opencore)
- [Steps](#steps)
  - [Step 1: Disable S0 sleep under Windows](#step-1-disable-s0-sleep-under-windows)
  - [Step 2: Dump and Modify the DSDT file (under Windows)](#step-2-dump-and-modify-the-dsdt-file-under-windows)
  - [Step 3: Download and Install Opencore](#step-3-download-and-install-opencore)
  - [Step 4: BIOS settings](#step-4-bios-settings)
  - [Step 5: Restart and Boot Using OpeCore](#step-5-restart-and-boot-using-opecore)
  - [Step 6: Modify Power Plans in Control Panel](#step-6-modify-power-plans-in-control-panel)
  - [Step 7: Turn off flashing lights while sleeping](#step-7-turn-off-flashing-lights-while-sleeping)
- [Extra explanations for Hackintosh users](#extra-explanations-for-hackintosh-users)

## Introduction

As we know Microsoft has been asking OEMs to deliberately block the option of S3 sleep mode on laptops (especially those released in recent years) and force the users to use S0 Sleep mode instead.

>S0 Sleep Mode keeps the computer in a so-called low-power state (not truly sleeping) so it can quickly wake-up and response to external events, but it consumes more power, and there have been many reports saying their laptop has been overheating insaide their backpacks, and even causing hardware damage.

>S3 Sleep Mode saves the system state to RAM, powers down most components for deeper power savings, but requires a slightly longer wake-up time (0.5-2 seconds) and the laptop won't automatically wake up when you lift the screen. You have to press a key or move your mouse to wake it up. But it prevents the laptop from overeating and also prevents it from falling into hibernation (which takes a whole precoess to boot up again). What's more, it only uses very few batter life (less than 10% a day) when it's sleeping.

## Why Using OpenCore
The operation involves modifying one of the ACPI tables, called the DSDT. OpenCore is a morden and safer bootloader initially designed for hackintoshing, as a successor of Clover. Same as Clover, The modification is not permanent - nothing is written to the firmware itself - but rather through a bootloader that loads the modified DSDT before Windows starts. As I tested, the modification have run extremely stable on my G14 2022 (6900HS 6700S) for over a month, so I suggest using OpenCore (and you can even try installing hackintosh on you 2022 model, you can refer to [this post](https://github.com/b00t0x/ROG-Zephyrus-G14-GA402-Hackintosh)).

## Steps
### Step 1: Disable S0 sleep under Windows
*This part is referred from the [clover s3 sleep tutorial](https://gist.github.com/raenye/d6645d7039a6136ccfb055e0f8517698):*

Run Command Prompt as Admin.

![Alt text](image-1.png)

To disable S0 sleep, type in the Admin Command Prompt:

```reg add HKLM\System\CurrentControlSet\Control\Power /v PlatformAoAcOverride /t REG_DWORD /d 0```

### Step 2: Dump and Modify the DSDT file (under Windows)
There are two tools you need to use:
1. [SSDTTime](https://github.com/corpnewt/SSDTTime) for dumping the DSDT table
2. [Xiasl]() for modifying the DSDT table

First you gonna download the __SSDTTime__ from the [link above](https://github.com/corpnewt/SSDTTime) and run the __SSDTTime.bat__ in that folder

![Alt text](Snipaste_2023-06-14_16-19-14-1.jpg)

Then type in "P" and press Enter, the output file would be located in the subfolder called Results

Download Xiasl Windows version (Xiasl-Win64.zip) from [this link](https://github.com/ic005k/Xiasl/releases)

Open the dunmped __DSDT.aml__ file using __Xiasl__ and the thins to be modified are on the right side

First find "__DefinitionBlock ("", "DSDT", 2, "ALASKA", "A M I ", 0x01072009)__" on top of the file

"__0x01072009__" is the version of file and could be different for different models. What you gonna do is increase the version, for example in this case, as hexadecimal is used here, you can change the last number fro "__9__" into "__A__" which looks like "__0x0107200A__"

![Alt text](Snipaste_2023-06-14_16-33-57-1.jpg)

Then press __ctrl+F__ and type "__(SS3, Zero)__", replace the "__Zero__" with "__One__" ----(SS3, One)

Then press __ctrl+F__ and type "__(XS3, Package (0x04)__", replace the "__XS3__" with "____S3___" ----(_S3, Package (0x04))

![Alt text](Snipaste_2023-06-14_16-37-55-1.jpg)

Then Press __Ctrl+M__ to compile the file, and the previous __DSDT.aml__ file would be overwritten in the same folder where it was, with the same name.

### Step 3: Download and Install Opencore
In this part, you gonna need to download [Expolrer++](https://explorerplusplus.com/) to help you move the bootloader files into your EFI partition. (64-bit suggested)

For your convenience, I've included a compitable OpenCore file in the project files. You can download the two folders (/OC and /BOOT). What you need to do is replace the __DSDT.aml__ file in __/OC/ACPI__ with your own modified one.

Then run Command Prompt as Admin again.

![Alt text](image-1.png)

Run the command below:

>mountvol R: /s

Then go to the Explorer++ folder and  __RIGHTCLICK__ and open Explorer++ __as Administrator__.

Go to __R:\EFI__ and put the 2 folders into it.

![Alt text](Snipaste_2023-06-14_17-20-14-1.jpg)

![Alt text](Snipaste_2023-06-14_17-21-58-1.jpg)

### Step 4: BIOS settings
Reboot the computer and keep clicking the ESC buttom when the boot sound beeing played, and chose enter setup. Then press F7 to enter advanced mode and disable Secure boot.

![Alt text](image-3.png)

Add the new Boot option:

![Alt text](bf9f7fd7fffe4dda4275197a917857b.jpg)

Set it as #1 boot priority

![Alt text](11eb3994da8361d72f44ab63bd68344.jpg)

### Step 5: Restart and Boot Using OpeCore
Reboot the computer and you would be booted into windows with S3 Sleep Mode enabled. You can use: 

>powercfg /a

>powercfg -a #If you're using powershell

In terminal to check if S3 sleep is enabled. Should be like this:

![Alt text](image-2.png)

### Step 6: Modify Power Plans in Control Panel
Search Control Panel using system search function and change to Big Icon Mode(I don't know exactly the name cuz my windows is not running in English, but should be on the top right)

Then go to power management? (The battery Icon)

Then go to change plan setting - Advanced settings - Sleep

Disable Hybrid sleep and Hibernation and the last thing (don't know its Egnlish name sorry)

Then your sleeping should be OK!

### Step 7: Turn off flashing lights while sleeping
*This part is referred from the [clover s3 sleep tutorial](https://gist.github.com/raenye/d6645d7039a6136ccfb055e0f8517698):*

During S3 sleep, the power on led on the left side double-blinks. If you're annoyed by the keyboard backlight double-blinking, this can be disabled in Armoury Crate->System->Lighting->Settings:

![Alt text](image-4.png)

![Alt text](image-5.png)

## Extra Explainations for Hackintosh Users
Everything's done from here. I'm only adding some explanations for hackintosh users who are worried about the SMBIOS influencing the windows.

I directly moded the EFI configured for G14 2022 hackintoshing by @b00t0x, but if you want to hackintosh the machine, I suggest you directly check [his project](https://github.com/b00t0x/ROG-Zephyrus-G14-GA402-Hackintosh) and mod the files provided there.

In [this](https://www.reddit.com/r/hackintosh/comments/lnh66w/windows_through_opencore_shows_as_macpro/?utm_source=share&utm_medium=ios_app&utm_name=iossmf) post, Some one provided the way to avoid SMBIOS settings in OC to impact windows. If you want to build your own OpenCore bootloader, you can refer to this.

>If you boot Windows from OC, to prevent OC from injecting SMBIOS values to Windows you have to act on these keys in config.plist:
- Kernel> Quirks> CustomSMBIOSGuid> True (default is False)
- PlatformInfo> UpdateSMBIOSMode> Custom (default is Create).
