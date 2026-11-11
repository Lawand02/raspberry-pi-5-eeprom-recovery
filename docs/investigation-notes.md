# Investigation notes

## Scope

This project documents two Raspberry Pi 5 observations associated with EEPROM firmware changes. It does not claim to have established a single root cause.

## Case 1 — boot failure

Reported observations:
- Raspberry Pi 5 stopped booting after an EEPROM update.
- Repeated eight-flash ACT LED pattern.
- No HDMI output.
- Linux did not start.
- Micron memory marking reported as `2CB77 D8CJC`.

The symptoms suggested a failure during early boot, potentially during SDRAM initialization. That interpretation should be treated as a diagnostic hypothesis unless confirmed by authoritative boot diagnostics.

## Case 2 — reported RAM capacity

A second Raspberry Pi 5 sold as a 4GB model was reported by Linux as having 2GB with newer EEPROM firmware. After reverting to the 2024-09-10 image, Linux reportedly showed 4GB.

The Reddit discussion below was useful as a community reference for this second observation. It is not proof that the two cases share the same cause.

## Hypotheses to investigate

- EEPROM/SDRAM compatibility or initialization changes.
- Differences in supported Micron SDRAM part numbers.
- Memory timing, power sequencing, or signal-integrity problems.
- A mismatch between physical DRAM and manufacturing/OTP information.
- Board repair or RAM replacement.

The RAM-replacement/OTP explanation is **not confirmed** without independent evidence from the boards.

## Important compatibility warning

The official `raspberrypi/rpi-eeprom` project explains that newer boards may depend on bootloader fixes for newer SDRAM part numbers, power-supply sequencing, or PMIC behaviour. An EEPROM image that predates the board can prevent reliable booting. The project also documents the manufacturing-version check and the `bootloader_allow_mfg_downgrade=1` override.

The override allows an older image past a version gate; it does not provide missing SDRAM support.

## What would strengthen the investigation

For each board, preserve:
- Board revision and serial-independent hardware identification details (do not publish serial numbers).
- Exact RAM chip marking and clear photographs, if available.
- EEPROM version before and after recovery.
- `/proc/device-tree/chosen/rpi-min-boot-ver` value, if the OS boots and the node exists.
- `vcgencmd bootloader_version` output.
- `sudo rpi-eeprom-update` output.
- `free -h` output.
- Exact ACT LED sequence and HDMI recovery-screen output.
- Repository commit hash and SHA256 hashes of the firmware binaries.

Avoid publishing device serial numbers, MAC addresses, Wi-Fi credentials, or other identifying secrets.
