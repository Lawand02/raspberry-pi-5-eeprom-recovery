# Raspberry Pi 5 EEPROM Recovery Investigation

A documented investigation into Raspberry Pi 5 boot failures and RAM-capacity reporting that appeared after EEPROM firmware changes.

This repository records a **board-specific recovery procedure** using a historical Raspberry Pi 5 EEPROM image dated **2024-09-10**, the Raspberry Pi 5 recovery executable available during the investigation, and the manufacturing-version downgrade configuration.

> [!CAUTION]
> **This is an experimental, board-specific procedure—not a general recommendation to downgrade Raspberry Pi 5 EEPROM firmware.**
>
> Raspberry Pi documentation warns that newer boards may require bootloader fixes for newer SDRAM parts, power sequencing, or PMIC behaviour. An EEPROM image older than the board's manufacturing requirements can leave a board unable to boot. Confirm the exact hardware, current bootloader, minimum bootloader version, and recovery method before proceeding. A downgrade override bypasses a version check; it does not add missing hardware support or make an incompatible image safe.

## Investigation summary

Two Raspberry Pi 5 observations motivated this work:

- **Case 1 — boot failure:** one board stopped booting after an EEPROM update. The reported symptoms included repeated eight-flash ACT LED behaviour, no HDMI output, and an apparent failure before Linux started. The board's Micron memory marking was reported as `2CB77 D8CJC`.
- **Case 2 — RAM capacity reporting:** a second board sold as a 4GB model was reported by Linux as having 2GB with one EEPROM version and 4GB after reverting to the 2024-09-10 EEPROM image.

These observations do not, by themselves, establish the root cause. Possible explanations include firmware/hardware compatibility, SDRAM initialization or timing, and a mismatch between installed memory and manufacturing information. RAM replacement and OTP mismatch remain hypotheses unless independently verified.

## Repository contents

- [Recovery procedure](docs/recovery-procedure.md) — commands and execution order.
- [Investigation notes](docs/investigation-notes.md) — observations, hypotheses, and limitations.
- [Verification checklist](docs/verification-checklist.md) — pre-flash and post-boot checks.
- [References](docs/references.md) — official documentation and related community reports.

## Before you start

1. Read the complete procedure and warnings.
2. Back up important data from the microSD card.
3. Identify the SD card device carefully. **Never assume it is `/dev/sda`.**
4. Record the board model/revision, RAM marking (if visible), current EEPROM version, and the board's minimum bootloader version if the OS boots.
5. Prefer Raspberry Pi Imager's official bootloader recovery options or Raspberry Pi support guidance when appropriate.
6. Do not proceed if the historical firmware's compatibility with your board is unknown and the board is needed for critical work.

## Tested environment and provenance

The command history for this investigation records:
- Raspberry Pi 5 bootloader family: BCM2712.
- Historical EEPROM image: `pieeprom-2024-09-10.bin`.
- Image version string observed: `5be4f304`.
- Recovery executable observed in the recorded experiment: dated `2026-10-30`.
- Recovery digest generated with the repository's `rpi-eeprom-digest` utility using `-c 2712`.

These are **historical observations from one procedure**, not a statement that the same recovery executable or image is currently the latest or appropriate for every Raspberry Pi 5.

## Disclaimer

This repository is provided for documentation and research. It is not affiliated with or endorsed by Raspberry Pi Ltd. Firmware recovery and downgrade operations carry a risk of making a board unbootable. You are responsible for verifying image provenance, compatibility, backups, and device selection.
