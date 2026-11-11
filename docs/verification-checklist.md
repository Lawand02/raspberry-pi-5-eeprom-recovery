# Verification checklist

## Before attempting recovery

- [ ] Confirm the exact Raspberry Pi model and board revision.
- [ ] Record the installed RAM marking if it is visible without unsafe disassembly.
- [ ] Record the current EEPROM version if the board can boot.
- [ ] Check the board's minimum bootloader version if available.
- [ ] Read current Raspberry Pi EEPROM documentation.
- [ ] Verify the provenance and date of `pieeprom.bin`.
- [ ] Verify the provenance and date of `recovery.bin`.
- [ ] Generate `pieeprom.sig` using the matching `rpi-eeprom-digest` utility.
- [ ] Back up the microSD card.
- [ ] Identify the correct SD card device using `lsblk`; do not assume `/dev/sda`.
- [ ] Understand that the manufacturing-version override does not guarantee hardware compatibility.

## During recovery

- [ ] Confirm all four files are in the root of the FAT boot partition.
- [ ] Confirm the digest comparison produces no differences.
- [ ] Run `sync` and unmount the card cleanly.
- [ ] Do not disconnect power while EEPROM programming is in progress.
- [ ] Record the recovery screen and ACT LED behaviour.

## After recovery

- [ ] Remove the recovery card before normal boot.
- [ ] Confirm the board boots into the OS.
- [ ] Record `vcgencmd bootloader_version`.
- [ ] Record `sudo rpi-eeprom-update`.
- [ ] Record `free -h`.
- [ ] Record the result separately for each board.
- [ ] Do not infer root cause from a successful downgrade alone.
