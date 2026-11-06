# ESP32-S3 Voltage and Resistance Meter

An embedded hardware project exploring a portable voltage and resistance meter built around an ESP32-S3-WROOM-1-N4, a 5 V LCD, and a custom PCB designed in Altium. This repository documents the design choices, revisions, and eventual test results.

> **Status: schematic development.** Battery power is a proposed revision. PCB fabrication, firmware operation, measurement accuracy, and battery runtime have not yet been demonstrated in this repository.

## Project goals

- Measure low positive DC voltages and resistance using conditioned ADC inputs.
- Display readings on a 5 V character LCD.
- Program the ESP32-S3 through its native USB interface.
- Develop a portable battery-powered version and document the engineering tradeoffs.

## Proposed battery architecture

```text
Protected 2S lithium battery pack (7.4 V nominal, 8.4 V full)
    |
    +-- Buck converter, 5 V ----> LCD and required 5 V circuitry
    |
    +-- Buck converter, 3.3 V --> ESP32-S3 and 3.3 V circuitry

USB-C D- ----------------------> GPIO19
USB-C D+ ----------------------> GPIO20
Common ground -----------------> Battery output, regulators, LCD, ESP32, USB
```

The saved local schematic predates this battery proposal and uses USB-derived power. The diagram above is a design target, not a description of a completed board.

## Design decisions and open questions

| Area | Direction | Still to verify |
| --- | --- | --- |
| Microcontroller | ESP32-S3-WROOM-1-N4 | Footprint, antenna keepout, boot/reset circuit |
| USB | Native USB on GPIO19/GPIO20; separate 5.1 kohm CC pull-downs | ESD protection, routing, and VBUS detection behavior for the selected programming/firmware mode |
| Power | Protected 2S pack with separate 5 V and 3.3 V buck converters | Exact pack, current budget, regulator parts, ripple, thermal behavior, and low-battery cutoff |
| Charging | Matching external 2S lithium charger as the initial approach | Pack protection, balancing, connector polarity, and charging specifications |
| LCD | 5 V character display; draft includes SN74AHCT245 signal translation | Exact display model, pinout, backlight current, and timing |
| Measurement | Draft voltage divider and resistance excitation networks | Input protection, ADC calibration, practical range, and measured error |

USB VBUS must remain separate from battery-generated 5 V unless a deliberate power-selection circuit is designed. A generic No ERC marker suppresses checks; it does not disconnect a pin. Single-cell charging boards such as BQ24074 or PowerBoost 1000C are not suitable chargers for a 2S pack.

## Documentation roadmap

- [ ] Add the reviewed Altium schematic source and a readable PDF export.
- [ ] Add a bill of materials with selected parts, footprints, and datasheet links.
- [ ] Record power calculations and the reasons for component choices.
- [ ] Run Altium electrical checks and resolve or explain remaining warnings.
- [ ] Complete PCB layout, including antenna clearance, USB routing, and regulator layout.
- [ ] Add firmware with build and flashing instructions.
- [ ] Bring up the power rails using a current-limited bench supply.
- [ ] Verify USB programming and LCD operation.
- [ ] Compare voltage and resistance readings against reference instruments.
- [ ] Record load current, regulator temperature, and battery runtime.
- [ ] Add board photos, a demonstration video, and a revision log.

## Planned repository organization

```text
hardware/   Altium sources, schematic exports, PCB outputs
firmware/   Source code and build instructions
docs/       Design decisions, power budget, test procedures and results
images/     Schematic views, PCB renders and prototype photos
```

These folders and hardware artifacts will be added as they are reviewed. This initial repository contains the project overview only.

## Validation record

No bench measurements are published yet. Each future test should record the hardware revision, firmware revision, setup, expected result, measured result, and any corrective action. Intended ranges and calculations are design targets until checked on hardware.

## Measurement limits

This is a low-voltage learning prototype, not a certified multimeter. The draft does not provide mains-rated protection or isolation. Resistance measurements are intended only for unpowered, discharged components. USB ground and measurement common are shared.

## References

- [ESP32-S3-WROOM-1 / WROOM-1U datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)
- [Espressif ESP32-S3 schematic checklist](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html)
- [Espressif self-powered USB guidance](https://docs.espressif.com/projects/esp-iot-solution/en/latest/usb/usb_overview/usb_device_self_power.html)
- [TI TPS62130 buck converter candidate](https://www.ti.com/product/TPS62130)
- [TI SN74AHCT245 LCD interface buffer](https://www.ti.com/lit/ds/symlink/sn74ahct245.pdf)
