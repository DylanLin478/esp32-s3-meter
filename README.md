# ESP32-S3 Voltage and Resistance Meter

Portable meter hardware in development using the **ESP32-S3-WROOM-1-N4**, a **3.3 V I²C OLED**, and a custom schematic in **Altium Designer**. Focus: analog signal conditioning, embedded interfaces, and battery power design.

**Status:** schematic development; no fabricated-board results, measurement accuracy, or battery-runtime claims yet.

## Design update — November 6, 2026

- Replaced the 5 V character LCD with a Winstar WEA012864LWPP3N00000 OLED, eliminating the planned 5 V display rail and level shifter.
- Changed the power architecture from a proposed 7.4 V pack with two bucks to a protected single-cell 3.7 V lithium battery and a 3.3 V TPS63021 buck-boost stage. This maintains regulation as battery voltage crosses 3.3 V; a matching external 1S charger is planned.
- Added separate voltage and resistance terminals, RC filtering, supply monitoring, and four navigation buttons.

## Schematic blocks

### ESP32 and USB programming

Native USB uses GPIO19 (D−) and GPIO20 (D+), 22 Ω series resistors, and separate 5.1 kΩ USB-C CC pull-downs. Boot/reset switches and local supply decoupling are included. USB programming and self-powered VBUS handling remain to be verified.

![ESP32-S3 and USB schematic](Screenshot%202026-11-06%20132502.png)

### OLED and user controls

Four-pin OLED interface: VCC, GND, SCL (GPIO9), SDA (GPIO8). Up, Down, Select, and Back use GPIO10–13 with planned firmware pull-ups and debounce.

![OLED connector and four navigation buttons](Screenshot%202026-11-06%20132506.png)

### Analog measurement circuits

| Circuit | Design approach | Intended range |
| --- | --- | --- |
| DC voltage, J3 / GPIO4 | 47 kΩ / 10 kΩ divider; 1 kΩ series resistor and 100 nF filter. Calculated ADC input at 12 V: 2.11 V. | 0–12 V DC |
| Resistance, J4 / GPIO5 | 33 kΩ excitation resistor; 1 kΩ series resistor and 100 nF filter. | Target 1–100 kΩ; unverified |
| Supply monitor / GPIO6 | Two equal 10 kΩ, 0.1% resistors divide the excitation rail in half for ADC measurement. | Nominal 1.65 V |

Planned calculations: `Vin = 5.7 × Vadc` and `Rx = 33 kΩ × Vohms / (Vsupply − Vohms)`, with `Vsupply ≈ 2 × Vhalf`. Firmware must use calibrated ADC voltages, account for resistor tolerances, and detect open/out-of-range inputs.

![Voltage, resistance, and supply monitoring circuits](Screenshot%202026-11-06%20132453.png)

Resistance measurements require unpowered, discharged components. Both measurement commons and USB ground are shared. Input protection is still incomplete; this prototype is for low-voltage DC and is not mains-rated.

### Single-cell battery power — draft under review

Protected 3.7 V lithium battery → JST-PH connector → TPS63021 buck-boost → 3.3 V rail. The draft includes a 1.5 µH inductor, input bypassing, and three 22 µF output capacitors.

![Draft TPS63021 battery power circuit](Screenshot%202026-11-06%20132510.png)

**Review finding:** this screenshot shows a 1 MΩ / 180 kΩ feedback divider, but the fixed-output **TPS63021 requires FB connected directly to VOUT**. Remove that divider and correct the connection before powering the circuit. Also verify VINA/EN supply wiring, PS/SYNC mode, inductor current rating, battery polarity, and capacitor effective values. See the [TI datasheet, pin functions and application guidance](https://www.ti.com/lit/ds/symlink/tps63021.pdf).

## Next validation steps

1. Resolve power-stage review findings, annotate components, and complete Altium electrical checks and BOM.
2. Complete PCB layout, including regulator current loops, USB routing, and ESP32 antenna clearance.
3. Implement USB programming, OLED menus, button debounce, and calibrated ADC measurements.
4. Bench-test regulation, ripple, measurement error against reference instruments, and battery runtime.

## Earlier revision

[Initial 5 V LCD and LDO schematic](Schematic-v1-LDO.png). A 2S battery with dual buck converters was considered before selecting the 3.3 V OLED and single-cell architecture above.
