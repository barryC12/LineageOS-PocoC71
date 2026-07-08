# LineageOS 23.2 — Poco C71 Bootloader Unlock & Flash Guide

> **Disclaimer:** I am not responsible for any damage to your device. Proceed at your own risk. Unlocking the bootloader **will wipe your phone**.

---

## Device Info

- **Device:** Poco C71
- **SoC:** Unisoc T7250
- **ROM:** LineageOS 23.2 (Android 16, GSI)

---

## Requirements

- Windows PC
- An unzipping program ([7-Zip](https://www.7-zip.org/) or [WinRAR](https://www.win-rar.com/))
- [Android Platform Tools (adb/fastboot)](https://developer.android.com/tools/releases/platform-tools)
- [Poco C71 USB Driver](https://gsmusbdriver.com/poco-c71)
- [Latest ADB Fastboot Installer (fastboot driver)](https://github.com/fawazahmed0/Latest-adb-fastboot-installer-for-windows/releases/download/v1.7/Latest-ADB-Installer.bat) — just run it and follow the prompts
- A high-quality USB-C data cable
- Some general knowledge of the process — read through the whole guide before starting

---

## Step 1 — Enable Developer Options & OEM Unlock

1. Go to **Settings → About Phone**
2. Tap **Build Number** repeatedly (about 5 times) until you're told you're a developer
3. Go back to **Settings → Additional Settings → Developer Options**
4. Enable **OEM Unlocking** and **USB Debugging**
5. Power off the phone

---

## Step 2 — Install Drivers

1. Download and run the [Poco C71 USB Driver](https://gsmusbdriver.com/poco-c71) installer
2. Run the `Latest-ADB-Installer.bat` file **as Administrator** and follow the on-screen instructions

---

## Step 3 — Unlock the Bootloader (CVE-2022-38694)

1. Download the unlock tool: [`ums9230e_Tecno_KL4.zip`](https://github.com/TomKing062/CVE-2022-38694_unlock_bootloader/releases/download/1.72/ums9230e_Tecno_KL4.zip)
2. Download the SPD driver: [`SPD_Driver_R4.20.4201.zip`](https://androiddatahost.com/wp-content/uploads/SPD_Driver_R4.20.4201.zip)
3. Unzip the SPD driver and run the installer `.exe` matching your PC's architecture (32-bit or 64-bit)
4. **Restart your PC** after the driver install finishes
5. Unzip `ums9230e_Tecno_KL4.zip` and run the `.bat` file inside it
6. With the phone powered off, hold **Power + Vol Down** for about 3 seconds — the screen should stay black
7. Keep holding **Vol Down** until the tool detects the device — do not let go early
8. Follow the prompts in the `.bat` window until it finishes

> ⚠️ This process wipes the phone. Once it's done, your bootloader is unlocked, but you'll need to set the phone up again and **re-enable Developer Options and USB Debugging**.

---

## Step 4 — Flash LineageOS 23.2

Download whichever build you want:

- **With GApps:** [`LineageOS-23.2-20260524-GAPPS-EXT4-GSI.7z`](https://sourceforge.net/projects/misterztr-gsi/files/LineageOS/Android%2016/LineageOS-23.2-20260524-GAPPS-EXT4-GSI.7z/download)
- **Vanilla (no GApps):** [`LineageOS-23.2-20260524-VANILLA-EXT4-GSI.7z`](https://sourceforge.net/projects/misterztr-gsi/files/LineageOS/Android%2016/LineageOS-23.2-20260524-VANILLA-EXT4-GSI.7z/download)

> All `adb`/`fastboot` commands below must be run **from inside your platform-tools folder** (open a terminal/Command Prompt there), which is why they're prefixed with `.\`.

1. Unzip the `.7z` file and copy the `.img` file into your platform-tools folder
2. Connect the phone and confirm ADB can see it:
  

```
.\adb -d devices
```

3. Accept the debugging prompt on the phone
4. Reboot to bootloader:
  

```
.\adb -d reboot bootloader
```

5. Reboot into fastboot mode:
  

```
.\fastboot reboot fastboot
```

6. Wipe and clear the product partition:
  

```
.\fastboot -w
.\fastboot delete-logical-partition product_a
```

7. Flash the system image (replace with the filename of the build you downloaded):
  

```
.\fastboot flash system LineageOS-23.2-20260524-GAPPS-EXT4-GSI.img
```

8. Reboot the phone. You should now be running LineageOS.

---

## Step 5 — Root with Magisk

> Run these from inside your platform-tools folder.

1. Download the boot image: [`magisk_boot.img`](https://github.com/barryC12/LineageOS-PocoC71/releases/download/BOOT/magisk_boot.img) and copy it into your platform-tools folder
2. Reboot to bootloader:
  

```
.\adb -d reboot bootloader
```

3. Flash it:
  

```
.\fastboot flash boot_a magisk_boot.img
```

4. Reboot back into the OS (`.\fastboot reboot`, or just do it manually)
5. You should now have a **Magisk** app in your app drawer — open it and follow the prompts to finish setup

---

## APN Notes (SIM users)

If you're using a SIM card, make sure your APN entry has **all** the required fields filled in — a missing field is the most common reason LTE won't be detected after flashing.

---

## Performance Fix — Persistent Swap File

Stock LineageOS performs pretty poorly on this phone out of the box. Setting up a swap file helps significantly.

> Root must be configured (via Magisk above) before doing this.

1. Install **Termux** from the Play Store, open it, and run each command one at a time:

```
su
dd if=/dev/zero of=/data/swapfile bs=1M count=4096
chmod 600 /data/swapfile
mkswap /data/swapfile
swapon /data/swapfile
```

2. To make the swap file persist across reboots:

```
su
mkdir -p /data/adb/service.d
cat > /data/adb/service.d/swapon.sh << 'EOF'
#!/system/bin/sh
# Wait a bit for /data to be fully mounted/decrypted
sleep 20
swapon /data/swapfile
EOF
chmod 755 /data/adb/service.d/swapon.sh
```

3. Get **EX Kernel Manager** and enable Performance mode by tapping the benchmark-looking icon in the upper corner
4. Turn off animations: go to **Developer Options** and turn off **Window animation scale**, **Transition animation scale**, and **Animator duration scale**

---

## What Works

- Boots and runs LineageOS 23.2
- Root via Magisk
- LTE (with a properly configured APN)
- Improved performance with swap enabled

## What Doesn't Work

> Tell me if you run into anything.

---

## Links

| Resource                     | Link                                                                                                                                                                     |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Poco C71 USB Driver           | [gsmusbdriver.com](https://gsmusbdriver.com/poco-c71)                                                                                                                    |
| Fastboot Installer            | [github.com/fawazahmed0](https://github.com/fawazahmed0/Latest-adb-fastboot-installer-for-windows/releases/download/v1.7/Latest-ADB-Installer.bat)                       |
| Bootloader Unlock Tool (CVE-2022-38694) | [github.com/TomKing062](https://github.com/TomKing062/CVE-2022-38694_unlock_bootloader/releases/download/1.72/ums9230e_Tecno_KL4.zip)                       |
| SPD Driver                    | [androiddatahost.com](https://androiddatahost.com/wp-content/uploads/SPD_Driver_R4.20.4201.zip)                                                                          |
| LineageOS 23.2 (GApps)        | [sourceforge.net](https://sourceforge.net/projects/misterztr-gsi/files/LineageOS/Android%2016/LineageOS-23.2-20260524-GAPPS-EXT4-GSI.7z/download)                       |
| LineageOS 23.2 (Vanilla)      | [sourceforge.net](https://sourceforge.net/projects/misterztr-gsi/files/LineageOS/Android%2016/LineageOS-23.2-20260524-VANILLA-EXT4-GSI.7z/download)                     |
| Magisk Boot Image             | [github.com/barryC12](https://github.com/barryC12/LineageOS-PocoC71/releases/download/BOOT/magisk_boot.img)                                                              |
| Platform Tools                | [developer.android.com](https://developer.android.com/tools/releases/platform-tools)                                                                                     |

---

## Final Note

Once you go through all of this — the bootloader unlock, flashing LineageOS 23.2, rooting, the swap file, EX Kernel Manager, and killing the animations — the Poco C71 genuinely turns into a beast. For a phone that costs under $100, you end up with a smooth, debloated, up-to-date Android 16 experience that punches way above its price tag. Well worth the effort.
