# Windows Boot Troubleshooting Guide

Two knowledge base articles covering common Windows startup failures: what the problem looks like, what causes it, and how to fix it using Windows Recovery Environment (WinRE) tools.

---

## Issue #1: "No Boot Device Found" Error

### Problem Description

When the computer is powered on, Windows never loads. After the manufacturer logo, the screen goes black or shows a message such as **"No boot device found"**, **"Boot device not found"**, or **"Reboot and select proper boot device."** The Windows logo and desktop never appear because the firmware (BIOS/UEFI) cannot find a bootable operating system, so startup stops before Windows Boot Manager can run.

Common symptoms: the same error returns after every restart, the keyboard still responds at the error screen, and the problem often appears right after a hardware change, power loss, failed update, or after plugging in a USB drive.

### Possible Causes

1. **Disconnected or failing storage drive.** A loose or damaged SATA data/power cable, a poorly seated NVMe drive, or a drive that is failing can keep the firmware from detecting the drive at startup. If the drive is not detected, there is nothing to boot from.
2. **Incorrect boot order or boot mode in BIOS/UEFI.** If a USB flash drive, external drive, or a new empty internal drive is first in the boot order, the computer tries to start from it, finds no operating system, and reports an error. A boot mode mismatch (UEFI vs. Legacy/CSM) can cause the same result: a drive partitioned as GPT must boot in UEFI mode, and a drive partitioned as MBR must boot in Legacy mode.
3. **Corrupted or missing boot files (BCD or EFI System Partition).** The Boot Configuration Data (BCD) is the database that tells Windows Boot Manager where Windows is installed and how to start it. If it is damaged, missing, or points to the wrong location (from disk errors, an interrupted update, power loss, malware, or partition changes), Windows cannot be found. On UEFI systems, a damaged EFI System Partition (ESP) causes the same problem.

### Troubleshooting Steps

1. **Remove external devices.** Unplug all USB flash drives, memory cards, external hard drives, and discs, then restart. This rules out the computer trying to boot from the wrong device.
2. **Confirm the drive is detected in BIOS/UEFI.** Power on and repeatedly tap the setup key (usually F2, F10, F12, Esc, or Delete, depending on the manufacturer).
   - On the main or storage screen, confirm the internal drive (HDD, SSD, or NVMe) is listed.
   - If it is not listed, power off and unplug the computer, then reseat the drive's data and power cables (or reseat the M.2 drive) and try a different port or cable. If the drive is still not detected, it has likely failed. Back up what you can and replace it.
3. **Check the boot order and boot mode.** In the Boot tab, set **Windows Boot Manager** (UEFI) or the internal Windows drive as the first boot device. Note whether the boot mode is **UEFI** or **Legacy/CSM**. Do not change it unless you know it was changed by mistake, since Windows 11 requires UEFI. Press F10 to save and restart.
4. **Boot into the Windows Recovery Environment (WinRE).** On a working PC, use Microsoft's Media Creation Tool to make a Windows installation/recovery USB. Then:
   - Plug the USB into the affected computer and boot from it (use the boot menu key, often F12, F9, or Esc).
   - Choose the language, click **Next**, then click **Repair your computer** in the bottom-left corner.
   - Go to **Troubleshoot > Advanced options**.
5. **Run Startup Repair.** Select **Troubleshoot > Advanced options > Startup Repair**. Let Windows run its diagnostics and repair damaged startup files automatically, then restart to see if Windows boots normally.
6. **Repair the boot files from the Command Prompt.** If Startup Repair fails, go back to **Troubleshoot > Advanced options > Command Prompt**. (If the drive is BitLocker-encrypted, you will be asked for the recovery key first.) The correct repair method depends on whether the disk uses UEFI/GPT or Legacy BIOS/MBR, so check that first.

   **6a. Identify the partition style and the Windows drive letter.**

   ```
   diskpart
   list disk
   exit
   ```

   - An asterisk (*) in the **Gpt** column means the disk is GPT (UEFI). No asterisk means MBR (Legacy BIOS).
   - In WinRE, the Windows drive is often not C:. Run `dir C:\`, `dir D:\`, and so on until you find the drive containing the **Windows** and **Users** folders. Use that letter wherever C: appears below.

   **6b. UEFI/GPT systems: rebuild the boot files in the EFI System Partition with bcdboot.**

   ```
   diskpart
   list disk
   select disk 0
   list volume
   select volume <number of the small FAT32 volume>
   assign letter=S:
   exit
   bcdboot C:\Windows /s S: /f UEFI
   ```

   - The EFI System Partition is the small (about 100-500 MB) **FAT32** volume. Replace `disk 0` with the disk that holds Windows if different.
   - Note: `bootrec /fixboot` often fails with "Access is denied" on UEFI systems, which is why `bcdboot` is used instead.

   **6c. Legacy BIOS/MBR systems: repair with bootrec.**

   ```
   bootrec /fixmbr
   bootrec /fixboot
   bootrec /scanos
   bootrec /rebuildbcd
   ```

   - `/fixmbr` writes a new master boot record, `/fixboot` writes a new boot sector, `/scanos` searches the drive for Windows installations, and `/rebuildbcd` rebuilds the BCD. If it finds Windows, press **Y** to add it to the boot list.
   - If `/rebuildbcd` reports 0 installations, back up and recreate the BCD store, then run `bootrec /rebuildbcd` again:

   ```
   bcdedit /export C:\BCD_Backup
   attrib c:\boot\bcd -h -r -s
   ren c:\boot\bcd bcd.old
   bootrec /rebuildbcd
   ```

7. **Restart and verify.** Type `exit`, remove the recovery USB, and restart. If Windows now loads, the issue is resolved. If the error persists, the drive may be failing: run `chkdsk C: /r` from the WinRE Command Prompt, back up important data, and replace the drive or reinstall Windows if needed.

---

## Issue #2: Blue Screen Stop Error During Startup (BSOD)

### Problem Description

The PC begins to start but crashes to a blue screen (Blue Screen of Death, or BSOD) showing a sad face, a message like "Your PC ran into a problem and needs to restart," and a stop code. This happens either as the Windows logo appears or before the sign-in screen. The computer may then restart and crash again at the same point, creating a restart loop or an Automatic Repair loop.

Common stop codes at startup include **INACCESSIBLE_BOOT_DEVICE**, **CRITICAL_PROCESS_DIED**, and **SYSTEM_THREAD_EXCEPTION_NOT_HANDLED**.

### Possible Causes

1. **Corrupted system files or a faulty Windows update.** A bad or incomplete update, or damaged core Windows files, can cause essential system processes to crash during startup (for example, CRITICAL_PROCESS_DIED).
2. **Faulty or incompatible hardware drivers.** A newly installed or updated driver, such as a graphics or storage controller driver, can conflict with Windows and crash the system before it finishes loading (for example, SYSTEM_THREAD_EXCEPTION_NOT_HANDLED).
3. **Disk errors or a storage configuration change.** File system corruption, bad sectors, or a failing drive can stop Windows from reading the files it needs. INACCESSIBLE_BOOT_DEVICE can also appear after a BIOS reset or update changes the SATA mode (for example, AHCI vs. RAID/Intel RST) so Windows can no longer access its boot drive.
4. **Failing hardware such as RAM.** Faulty memory can corrupt data while Windows loads and trigger random stop errors during startup.

### Troubleshooting Steps

1. **Record the stop code.** Write down or photograph the stop code and note any recent changes (new driver, update, hardware, or BIOS change). This guides which of the steps below to prioritize.
2. **Force the PC into the recovery menu.** Turn on the PC. As soon as the Windows logo or spinning dots appear, press and hold the power button for 5-10 seconds to force it off. Repeat 2-3 times until you see **"Preparing Automatic Repair."** Then choose **Advanced options** to open the blue recovery screen (WinRE). If this does not work, boot from a recovery USB as in Issue #1.
   - If the stop code is INACCESSIBLE_BOOT_DEVICE and the BIOS was recently reset or updated, enter BIOS/UEFI and make sure the SATA mode (AHCI vs. RAID) matches what it was before.
3. **Try System Restore.** Go to **Troubleshoot > Advanced options > System Restore** and choose a restore point from before the problem began. This reverses recent system changes without touching personal files. (Skip this if no restore points exist.)
4. **Start in Safe Mode.** Go to **Troubleshoot > Advanced options > Startup Settings > Restart**. When the list appears, press **4** (or F4) for Safe Mode, or **5** (or F5) for Safe Mode with Networking. Safe Mode loads only the minimum drivers and services needed. If Windows starts here, the cause is likely a driver, update, or software problem.
5. **Roll back or remove the problem driver.** Once Windows loads in Safe Mode:
   - Right-click the Start button and open **Device Manager**.
   - Find the device you recently updated (often under **Display adapters** or **Storage controllers**).
   - Right-click it, choose **Properties**, open the **Driver** tab, and click **Roll Back Driver**. If that is unavailable, choose **Uninstall Device**, then restart so Windows reinstalls a default driver.
6. **Uninstall the latest Windows update.** Return to the recovery screen and select **Troubleshoot > Advanced options > Uninstall Updates**. Choose **Uninstall latest quality update** (or **feature update**). This removes the recent update without deleting personal files. Note that feature updates can only be uninstalled for a limited time after installation (10 days by default).
7. **Scan and repair the drive and system files.** Open **Troubleshoot > Advanced options > Command Prompt**. The Windows drive is often not C: in WinRE, so confirm the letter first by running `dir C:\` (or D:, E:, etc.) until you see the **Windows** folder. Then run:

   ```
   chkdsk C: /r
   sfc /scannow /offbootdir=C:\ /offwindir=C:\Windows
   ```

   - `chkdsk /r` scans for bad sectors and file system errors and recovers readable data. It can take a long time on large drives.
   - `sfc /scannow` with the offline switches finds and replaces corrupted Windows system files.
   - Close the window and restart the PC.
8. **If the crashes continue.** Test the memory with Windows Memory Diagnostic (run `mdsched.exe` from Safe Mode, if you can reach it). If everything else fails, back up your files and use **Troubleshoot > Reset this PC** (choose **Keep my files**) or reinstall Windows.
