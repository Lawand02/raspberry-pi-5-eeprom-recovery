# Raspberry Pi 5 EEPROM recovery procedure

This document organizes the command sequence used during one Raspberry Pi 5 EEPROM investigation.

> [!CAUTION]
> This procedure uses a historical EEPROM image dated 2024-09-10. Newer Raspberry Pi 5 hardware may require newer SDRAM initialization or other bootloader support. Raspberry Pi's own `rpi-eeprom` documentation warns that installing an image older than a board's minimum manufacturing bootloader version can prevent reliable booting.
>
> The `bootloader_allow_mfg_downgrade=1` setting bypasses the manufacturing-version downgrade check. It does **not** guarantee compatibility with the installed RAM or other hardware.
>
> Do not use this procedure as a routine update or downgrade guide. Verify the exact board and follow current official documentation.

## 1. Install dependencies and clone the official repository

Run on a Linux computer:

```bash
sudo apt update
sudo apt install -y git python3

mkdir -p ~/EEPROM
cd ~/EEPROM

git clone https://github.com/raspberrypi/rpi-eeprom.git
```

If the repository already exists:

```bash
cd ~/EEPROM/rpi-eeprom
git pull
```

For reproducibility, record the repository commit used:

```bash
cd ~/EEPROM/rpi-eeprom
git rev-parse HEAD
```

## 2. Locate and identify the historical EEPROM image

```bash
cd ~/EEPROM/rpi-eeprom

find . -iname '*2024-09-10*'
```

Expected path in the recorded repository checkout:

```text
firmware-2712/old/default/pieeprom-2024-09-10.bin
```

Inspect the embedded version/date:

```bash
strings firmware-2712/old/default/pieeprom-2024-09-10.bin \
  | grep -E 'VERSION|DATE'
```

Observed output:

```text
VERSION:5be4f304
DATE: 2024/09/10
```

If the path or output differs, stop and inspect the repository history rather than assuming the image is identical.

## 3. Create a working directory and copy the EEPROM image

```bash
mkdir -p ~/EEPROM/rpi5-sept10

cp firmware-2712/old/default/pieeprom-2024-09-10.bin \
  ~/EEPROM/rpi5-sept10/pieeprom.bin
```

Verify:

```bash
ls -lh ~/EEPROM/rpi5-sept10/pieeprom.bin

strings ~/EEPROM/rpi5-sept10/pieeprom.bin \
  | grep -E 'VERSION|DATE'
```

## 4. Obtain the Raspberry Pi 5 recovery executable

Update the official repository:

```bash
cd ~/EEPROM/rpi-eeprom
git pull
```

Copy the recovery executable from the BCM2712 firmware directory:

```bash
cp firmware-2712/default/recovery.bin \
  ~/EEPROM/rpi5-sept10/recovery.bin
```

Inspect it:

```bash
strings ~/EEPROM/rpi5-sept10/recovery.bin \
  | grep -E 'VERSION|DATE'
```

In the recorded experiment, the historical EEPROM image was dated 2024-09-10 while the recovery executable was dated 2026-10-30. Record the actual output from your own checkout; `git pull` means the file may change over time.

## 5. Generate the EEPROM digest/signature file

Do not substitute an arbitrary checksum command for the Raspberry Pi utility. Generate the file using the `rpi-eeprom-digest` version in the checked-out repository:

```bash
cd ~/EEPROM/rpi-eeprom

./rpi-eeprom-digest \
  -c 2712 \
  -i ~/EEPROM/rpi5-sept10/pieeprom.bin \
  -o ~/EEPROM/rpi5-sept10/pieeprom.sig
```

Inspect the generated file:

```bash
cat ~/EEPROM/rpi5-sept10/pieeprom.sig
```

The recorded BCM2712 workflow included a `target-soc: 2712` field. Treat the output produced by the matching utility as authoritative; do not hand-edit the file.

## 6. Configure the manufacturing-version downgrade override

Create the recovery configuration:

```bash
printf 'bootloader_allow_mfg_downgrade=1\n' \
  > ~/EEPROM/rpi5-sept10/config.txt
```

Verify:

```bash
cat ~/EEPROM/rpi5-sept10/config.txt
```

Expected:

```ini
bootloader_allow_mfg_downgrade=1
```

This is a compatibility override, not a compatibility fix. Only use it when you understand the risk and have verified the target image and board requirements.

## 7. Verify the recovery package

```bash
cd ~/EEPROM/rpi5-sept10

ls -lh

sha256sum pieeprom.bin

file recovery.bin pieeprom.bin

cat config.txt

cat pieeprom.sig
```

Expected directory layout:

```text
rpi5-sept10/
├── config.txt
├── pieeprom.bin
├── pieeprom.sig
└── recovery.bin
```

## 8. Identify the microSD card carefully

List block devices:

```bash
lsblk -o NAME,SIZE,MODEL,TRAN,FSTYPE,LABEL,MOUNTPOINTS
```

Example output from the investigation:

```text
sda       29.7G
├─sda1    512M   vfat
└─sda2      2G   ext4
```

> [!WARNING]
> The example device `/dev/sda` is not universal. Your card may be `/dev/mmcblk0`, `/dev/sdb`, or another device. Identify it by size, model, transport, and partitions. Selecting the wrong device can damage your computer's data.

Set the device and partition variables only after identifying the correct card. For the example above:

```bash
SD_DEVICE=/dev/sda
BOOT_PARTITION=/dev/sda1
ROOT_PARTITION=/dev/sda2
```

Unmount existing partitions (adjust the names if necessary):

```bash
sudo umount "$BOOT_PARTITION" 2>/dev/null || true
sudo umount "$ROOT_PARTITION" 2>/dev/null || true
```

Create a mount point and mount the FAT boot partition:

```bash
sudo mkdir -p /mnt/rpi-recovery
sudo mount "$BOOT_PARTITION" /mnt/rpi-recovery

ls -la /mnt/rpi-recovery
```

This procedure uses the existing FAT boot partition. It does not require formatting the entire card in the recorded workflow. Back up the card first and confirm the recovery layout against current official instructions.

## 9. Copy and verify the recovery files

```bash
sudo cp ~/EEPROM/rpi5-sept10/recovery.bin \
  /mnt/rpi-recovery/

sudo cp ~/EEPROM/rpi5-sept10/pieeprom.bin \
  /mnt/rpi-recovery/

sudo cp ~/EEPROM/rpi5-sept10/pieeprom.sig \
  /mnt/rpi-recovery/

sudo cp ~/EEPROM/rpi5-sept10/config.txt \
  /mnt/rpi-recovery/
```

List the copied files:

```bash
ls -lh /mnt/rpi-recovery/
```

Regenerate the digest from the copied EEPROM image and compare it with the copied digest:

```bash
cd ~/EEPROM/rpi-eeprom

./rpi-eeprom-digest \
  -c 2712 \
  -i /mnt/rpi-recovery/pieeprom.bin \
  -o /tmp/verify-pieeprom.sig

diff -u /tmp/verify-pieeprom.sig \
  /mnt/rpi-recovery/pieeprom.sig
```

Expected: `diff` prints no output when the files match.

Flush writes and unmount:

```bash
sync
sudo umount /mnt/rpi-recovery
```

The official documentation describes the roles of `recovery.bin`, `pieeprom.bin`/`pieeprom.upd`, and `pieeprom.sig`. See [official EEPROM documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-boot-eeprom).

## 10. Perform the recovery

1. Disconnect Raspberry Pi power.
2. Insert the prepared microSD card.
3. Connect HDMI to HDMI0 if you want to observe recovery output.
4. Power on the Raspberry Pi.
5. Observe the recovery screen and ACT LED.
6. Allow the recovery operation to complete; do not interrupt power during programming.
7. Disconnect power after the operation has completed.
8. Remove the recovery microSD card.
9. Power on again and observe the boot behaviour.

When the update image is named `pieeprom.bin`, the documented recovery behaviour is to stop after programming rather than rename `recovery.bin` and continue booting. A green recovery screen indicates the recovery process ran; it does not alone prove that the board will boot successfully with the programmed firmware.

If recovery reports an error, record the exact HDMI/LED indication. Do not repeatedly try older firmware images without investigating compatibility.

## 11. Install or boot a normal Raspberry Pi OS card

Use Raspberry Pi Imager to prepare a normal OS card if needed.

Suggested selection:
- Raspberry Pi 5
- Raspberry Pi OS Lite (64-bit), if a headless setup is desired
- Configure username/password and SSH as appropriate

Boot with the normal OS card after removing the recovery card.

## 12. Verify the installed bootloader and RAM

Once Raspberry Pi OS boots:

```bash
sudo rpi-eeprom-update
vcgencmd bootloader_version
free -h
```

Optional supplementary command:

```bash
vcgencmd get_mem arm
```

Record all output, including the date/version and the OS-reported total memory. `free -h` is the primary Linux-side check for total usable memory; `vcgencmd get_mem arm` reports the ARM memory allocation and should not be treated as a replacement for the Linux memory total.

## 13. Prevent automatic EEPROM updates while investigating

Inspect the configuration:

```bash
cat /etc/default/rpi-eeprom-update
```

The release channel can be selected in this file using `FIRMWARE_RELEASE_STATUS`. For example:

```ini
FIRMWARE_RELEASE_STATUS="critical"
```

This selects a release stream; it does **not** freeze the installed EEPROM version.

For a temporary investigation freeze, the current Raspberry Pi documentation recommends masking the update service:

```bash
sudo systemctl mask rpi-eeprom-update
```

Verify:

```bash
systemctl is-enabled rpi-eeprom-update
```

Expected:

```text
masked
```

Re-enable automatic updates when ready:

```bash
sudo systemctl unmask rpi-eeprom-update
```

Do not manually install another EEPROM until you have established which firmware versions are compatible with the board. Consult the current official documentation before changing update policy.

## Results log

Record the outcome for each board:

| Field | Value |
|---|---|
| Board model/revision | |
| RAM part marking | |
| EEPROM version before recovery | |
| Historical EEPROM image | 2024-09-10 |
| Recovery executable date | |
| Recovery screen result | |
| EEPROM version after recovery | |
| `free -h` output | |
| Boot result | |
| Notes / errors | |

Do not describe a board as recovered unless it boots normally and the installed EEPROM version and usable RAM have been checked.
