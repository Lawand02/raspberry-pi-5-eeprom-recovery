# References

## Official Raspberry Pi sources

- [Raspberry Pi EEPROM documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-boot-eeprom)
  - Recovery executable and update file roles.
  - Behaviour differences between `pieeprom.bin` and `pieeprom.upd`.
  - Automatic EEPROM update service controls.
- [Raspberry Pi `rpi-eeprom` repository](https://github.com/raspberrypi/rpi-eeprom)
  - Official EEPROM images, utilities, release history, and compatibility notes.
- [Raspberry Pi `rpi-eeprom` README](https://github.com/raspberrypi/rpi-eeprom/blob/master/README.md)
  - Minimum bootloader version (`MIN_BOOT_VER`) and manufacturing-version downgrade behaviour.

## Related reports

- [Raspberry Pi EEPROM Issue #761](https://github.com/raspberrypi/rpi-eeprom/issues/761)
  - A related report involving Raspberry Pi 5 boot failure after an EEPROM update.
- [Reddit: Raspberry Pi 5 4GB shows total RAM as 2GB](https://www.reddit.com/r/raspberry_pi/comments/1tcfc4p/raspberry_pi_5_4gb_shows_total_ram_as_2gb_in/)
  - Community discussion that helped guide the investigation of the second board's RAM-capacity reporting.

## Notes

Community reports are useful leads, not authoritative confirmation of root cause. Always distinguish reproduced observations from hypotheses.
