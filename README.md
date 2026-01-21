# Hard Drive Docking Station Setup Guide for Ubuntu Linux

---

📄 Author: **Ouail Zakary**  
- 📧 Email: [Ouail.Zakary@oulu.fi](mailto:Ouail.Zakary@oulu.fi)  
- 🔗 ORCID: [0000-0002-7793-3306](https://orcid.org/0000-0002-7793-3306)  
- 🌐 Website: [Personal Webpage](https://cc.oulu.fi/~nmrwww/members/Ouail_Zakary.html)  
- 📁 Portfolio: [GitHub Portfolio](https://ozakary.github.io/)

---

This guide explains how to set up and use hard drives with a USB docking station on Ubuntu Linux. This process was tested with a dual-bay hard drive docking station and two 4TB Seagate drives.

## Table of Contents
- [Overview](#overview)
- [Hardware Setup](#hardware-setup)
- [Initial Detection](#initial-detection)
- [Common Issues](#common-issues)
- [Setting Up a New Drive](#setting-up-a-new-drive)
- [Final Verification](#final-verification)
- [Daily Usage](#daily-usage)
- [Troubleshooting](#troubleshooting)

---

## Overview

**What you need:**
- Hard drive docking station with USB connection
- One or more hard drives (SATA)
- Ubuntu Linux system (tested on Ubuntu 22.04)
- Administrator (sudo) access

**What this does NOT require:**
- The "cloning" feature mentioned in the docking station manual is OPTIONAL and NOT needed for normal storage use
- You only need cloning if you want to duplicate one drive to another

---

## Hardware Setup

### Step 1: Connect the Docking Station

1. Connect the USB cable from the docking station to your laptop
2. Insert your hard drive(s) into the docking station slots (HDD A and/or HDD B)
3. Connect the AC/DC power adapter to the docking station and wall socket
4. The drives should power on (you may see indicator lights)

**Important:** Do NOT use the cloning feature unless you specifically want to duplicate one drive to another. For normal storage use, just connect and use the drives.

---

## Initial Detection

### Check if Linux Detects Your Drive(s)

Open a terminal and run:

```bash
lsblk
```

**What to look for:**
- New devices named `sdb`, `sdc`, `sdd`, etc. (depends on what's already connected)
- The size should match your hard drive capacity (e.g., 3.6T for a 4TB drive)

**Example output:**
```
NAME          SIZE TYPE MOUNTPOINT
sda         931.5G disk 
└─sda1      931.5G part /media/user/existing_drive
sdc           3.6T disk                              ← New 4TB drive
nvme0n1     953.9G disk 
├─nvme0n1p1   512M part /boot/efi
└─nvme0n1p2   953G part /
```

### Check for Existing Data or RAID Configuration

```bash
sudo fdisk -l /dev/sdc
```

Replace `sdc` with your actual device name.

If you see partitions listed, the drive may have data. If you see "RAID" or "md" devices, see the [RAID Cleanup](#raid-cleanup) section below.

---

## Common Issues

### Issue: Drive Shows RAID Configuration

If your drive was previously used in a RAID array (common with enterprise drives), you'll see inactive RAID devices:

```bash
cat /proc/mdstat
```

**Example problematic output:**
```
md126 : inactive sdc[0]
      3906469888 blocks super external:/md127/1
```

**Solution:** See the [RAID Cleanup](#raid-cleanup) section below.

---

## Setting Up a New Drive

This section covers setting up a drive from scratch (or after wiping an existing drive).

### RAID Cleanup (if applicable)

If the drive has RAID metadata, clean it up first:

```bash
# Stop any inactive RAID arrays
sudo mdadm --stop /dev/md126
sudo mdadm --stop /dev/md127

# Wipe RAID metadata from the drive
sudo mdadm --zero-superblock /dev/sdc
```

Replace `sdc` with your actual device name.

### Step 1: Create a Partition Table

```bash
sudo fdisk /dev/sdc
```

**In the fdisk prompt, type these commands:**

1. Type `g` and press Enter (creates GPT partition table - required for drives >2TB)
2. Type `n` and press Enter (creates new partition)
3. Press Enter (accept default partition number)
4. Press Enter (accept default first sector)
5. Press Enter (accept default last sector - uses entire disk)
6. Type `w` and press Enter (write changes and exit)

**Expected output:**
```
Created a new GPT disklabel (GUID: ...)
Created a new partition 1 of type 'Linux filesystem' and of size 3.6 TiB.
The partition table has been altered.
```

### Step 2: Format the Partition

```bash
sudo mkfs.ext4 /dev/sdc1
```

**Note:** This will take a few minutes for large drives. You'll see progress messages.

**Expected output:**
```
Creating filesystem with 976754385 4k blocks and 244195328 inodes
Filesystem UUID: 8a84db63-36b2-44ec-8e68-8a3d6b47e3bb
...
Writing inode tables: done
Creating journal: done
Writing superblocks and filesystem accounting information: done
```

### Step 3: Let Ubuntu Auto-Mount the Drive

After formatting, **safely remove and re-plug the USB cable**. Ubuntu will automatically detect and mount the drive with its UUID as the folder name.

Check where it's mounted:

```bash
lsblk
```

You should see something like:
```
sdc           3.6T disk 
└─sdc1        3.6T part /media/username/8a84db63-36b2-44ec-8e68-8a3d6b47e3bb
```

### Step 4: Create Friendly Shortcuts (Optional but Recommended)

The UUID names are long and hard to remember. Create symbolic links for easier access:

```bash
# For the first drive
ln -s /media/username/8a84db63-36b2-44ec-8e68-8a3d6b47e3bb ~/hdd1

# For the second drive (if you have one)
ln -s /media/username/3fda220b-c107-4b75-97bc-05e5c3a5335c ~/hdd2
```

Replace:
- `username` with your actual Ubuntu username
- The UUID strings with the actual UUIDs from your `lsblk` output

Now you can access your drives as `~/hdd1` and `~/hdd2`!

---

## Final Verification

### Test Drive Access

```bash
# Test writing to the drive
echo "Test file" > ~/hdd1/test.txt

# Test reading from the drive
cat ~/hdd1/test.txt

# Check available space
df -h ~/hdd1
```

**Expected output:**
```
Test file
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc1       3.6T   36K  3.4T   1% /media/username/...
```

---

## Daily Usage

### Accessing Your Drives

Simply use the shortcuts you created:

```bash
# Copy files to the drive
cp my_data.tar.gz ~/hdd1/

# List files on the drive
ls ~/hdd1/

# Check disk usage
df -h ~/hdd1
```

### Safely Removing Drives

**Always safely eject before unplugging:**

1. **GUI method:** Right-click the drive icon and select "Safely Remove" or "Eject"
2. **Command line method:**
   ```bash
   # Unmount the drive
   sudo umount ~/hdd1
   
   # Or unmount by device
   sudo umount /dev/sdc1
   ```

3. Wait for the command to complete
4. Unplug the USB cable or power off the docking station

### Re-connecting Drives

When you plug the drives back in:
- Ubuntu will auto-mount them with the same UUID names
- Your symbolic links (`~/hdd1`, `~/hdd2`) will work automatically
- The device names (`sdc`, `sdd`) may change, but the mount points stay consistent

---

## Troubleshooting

### Drive Not Detected

**Check physical connections:**
```bash
lsusb  # Should show the docking station
dmesg | tail -30  # Check for USB device messages
```

**Check if drive is detected but not mounted:**
```bash
lsblk
sudo fdisk -l
```

### I/O Error When Accessing Drive

This usually means the drive connection was interrupted. Try:

```bash
# Unmount the problematic drive
sudo umount /media/username/UUID-name

# Check dmesg for errors
sudo dmesg | tail -50 | grep error

# Safely remove and re-plug the USB cable
# The drive should re-mount automatically
```

### Device Name Changed After Reboot

This is normal! Device names (`sdc`, `sdd`) can change between boots or when you unplug/replug drives. This is why we use:
- **UUID-based mount points** (handled automatically by Ubuntu)
- **Symbolic links** (`~/hdd1`) for easy access

Your data is safe - only the device name changed, not the actual drive.

### Drive Shows as "Read-Only"

Check ownership and permissions:

```bash
ls -la /media/username/UUID-name/

# If owned by root, change ownership
sudo chown username:users /media/username/UUID-name/
```

Replace `username` with your actual username. You can check your username with:
```bash
whoami
id
```

### Multiple Drives in Docking Station

If using both bays in a dual-bay docking station:
- Each drive appears as a separate device (`sdc`, `sdd`, etc.)
- Each gets its own UUID-based mount point
- Create separate shortcuts for each: `~/hdd1`, `~/hdd2`, etc.
- You can access both drives simultaneously

---

## Useful Commands Reference

### Check all drives and their mount points:
```bash
lsblk
df -h | grep -v loop
```

### Check drive health (requires smartmontools):
```bash
sudo apt install smartmontools
sudo smartctl -a /dev/sdc
```

### See what's using a drive:
```bash
lsof +D ~/hdd1
```

### Check disk usage:
```bash
du -sh ~/hdd1/*  # Size of each folder/file
df -h ~/hdd1     # Total space used/available
```

---

## Summary

**Quick setup for new drives:**

1. Connect docking station hardware
2. Check detection: `lsblk`
3. Clean RAID if needed: `sudo mdadm --zero-superblock /dev/sdc`
4. Partition: `sudo fdisk /dev/sdc` → `g`, `n`, Enter×3, `w`
5. Format: `sudo mkfs.ext4 /dev/sdc1`
6. Unplug and re-plug USB (Ubuntu auto-mounts)
7. Create shortcut: `ln -s /media/username/UUID ~/hdd1`
8. Test: `echo "test" > ~/hdd1/test.txt`

**Daily usage:**
- Access via `~/hdd1` and `~/hdd2`
- Always safely eject before unplugging
- UUID names stay consistent across reboots

---

## Notes

- Tested with Ubuntu 22.04 LTS
- Works with dual-bay USB 3.0 docking stations
- Suitable for 4TB Seagate ST4000NM drives (and other large drives)
- UUID-based mounting ensures consistency across reboots
- No need to use the docking station's "cloning" feature for normal storage use

---
For further details, please refer to the respective folders or contact the author via the provided email.
