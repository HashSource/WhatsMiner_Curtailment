# WhatsMiner Dynamic Power Curtailment

A practical guide for implementing dynamic power adjustment on WhatsMiner ASICs without rebooting.

## Prerequisites

- **Firmware Requirements:**
  - Minimum: 20200801 (for Power Fast Boot support)
  - Recommended: 20241011 or newer (for full feature set)
  - [Download Latest WhatsMinerTool](https://www.whatsminer.com/src/views/firmware-download.html#Tool)
  - [Download Latest Firmware](https://www.whatsminer.com/src/views/firmware-download.html#Firmware)
- **Hardware Compatibility:** M30 series and newer models only
- **Power Fast Boot:** **MUST** be enabled for dynamic adjustment without rebooting

## Quick Start

### 1. Enable Power Fast Boot (btminer Fast Boot)

Navigate to Miner Config and enable **Power Fast Boot** (one-time reboot required).

**Technical Detail:** This feature (API: `enable_btminer_fast_boot`) prioritizes rapid power target achievement over hashrate stabilization, allowing the miner to reach full load within 1 minute at startup.

![Power Fast Boot Configuration](assets/images/power_fast_boot_config.jpg)

### 2. Configure Upfreq Speed (Adjust Upfreq Speed)

Controls frequency tuning speed during startup (range: 0-10). Choose based on your needs:

- **0:** Normal frequency search speed (default, most stable)
- **1-2:** Maximum efficiency, slowest tuning (base load operations)
- **5-6:** Balanced for daily curtailment (8-10 min to full hashrate)
- **8-10:** Fastest frequency ramp (~6-7 min to full hashrate), 2-3% efficiency loss

**Technical Note:** Higher values accelerate frequency search but may impact stability. The faster the speed, the greater the final hash rate and power deviation. Cannot be used simultaneously with Fast Boot mode.

![Upfreq Speed Configuration](assets/images/upfreq_speed_config.jpg)

### 3. Adjust Power Dynamically

Navigate to **Adjust Power** and select between:

- **Normal Mode (set_power_pct_v2):** Gradual adjustment spanning several minutes with minimal hashrate decline
- **Fast Mode (set_power_pct):** Dynamic adjustment within a single second, only approximate power percentage reached
- **Wattage Mode (set_power):** Direct wattage value setting for precise power control

![Power Adjustment Interface](assets/images/power_adjustment_interface.jpg)

Enter target wattage directly (e.g., 3700W → 3200W → 3700W). The miner adjusts without rebooting.

**Important:** Power adjustments are percentage-based internally but should be specified as absolute wattage in the interface.

## Common Curtailment Scenarios

### Daily Peak Shaving (2-6pm)

- Power Fast Boot: ON
- Upfreq Speed: 5-6
- Mode: Normal
- Strategy: 60% power during peak hours

### Grid Emergency (<5 min response)

- Power Fast Boot: ON
- Upfreq Speed: 10
- Mode: Fast (if <2 min required)
- Strategy: Drop to minimum power immediately

### Solar/Renewable Integration

- Power Fast Boot: ON
- Upfreq Speed: 6-8
- Mode: Normal
- Strategy: Match generation curve

### Heat Management (Summer)

- Power Fast Boot: ON
- Upfreq Speed: 3-5
- Strategy: Reduce power with temperature rise

## Quick Reference

| Response Time | Upfreq | Mode    | Use Case           |
| ------------- | ------ | ------- | ------------------ |
| <2 minutes    | 10     | Fast    | Emergency only     |
| <5 minutes    | 8-10   | Normal  | Grid response      |
| 10-15 minutes | 5-6    | Normal  | Daily curtailment  |
| Precise power | Any    | Wattage | Exact power limits |
| Any           | 3-5    | Normal  | Weekly adjustments |

## Key Points

- **Without Power Fast Boot:** Miner drops to 0W and restarts (sudden load release)
- **With Power Fast Boot:** Smooth power transitions, no mining interruption
- **Normal Mode (set_power_pct_v2):** Adjustment spans several minutes, minimal hashrate decline, well-suited for frequent power adjustments
- **Fast Mode (set_power_pct):** Sub-second adjustment, only approximate power percentage reached, advisable for temporary use only
- **Wattage Mode (set_power):** Direct wattage setting for precise power control without percentage calculations
- **Power Limit vs Power Percent:** Use percentage for temporary adjustments, limit for long-term operation

## Restoring to Normal Operation

When you need to restore the miner back to tuning for best hashrate/efficiency:

### Method 1: Restore Power PCT (API: restore_power_pct)

- Restores configurations made via set_power_pct or set_power_pct_v2
- No reboot required
- Returns to previous stable mining power

### Method 2: Restore Miner Setting (Full Reset)

1. Navigate to the **Restore Miner Setting** option
2. Click the restore button (machine will reboot)

![Restore Miner Button](assets/images/restore_miner_button.png)

![Restore Miner Setting Interface](assets/images/restore_miner_setting_button.png)

**Important Notes:**

- Method 1 is preferred for quick restoration after percentage-based adjustments
- Method 2 requires full reboot and clears tuning cache files
- After Method 2, expect extended startup time as the miner re-tunes from scratch

## Troubleshooting

### Common Issues

| Issue                                   | Cause                            | Solution                                                 |
| --------------------------------------- | -------------------------------- | -------------------------------------------------------- |
| **Miner still reboots**                 | Power Fast Boot not enabled      | Enable Power Fast Boot in settings (requires one reboot) |
|                                         | Old firmware                     | Update to firmware ≥20200801 (M30+ only)                 |
| **Percentage adjustment causes reboot** | API using percentage values      | Always use absolute wattage (e.g., 3200 not 80%)         |
| **No power adjustment effect**          | Incompatible hardware            | Feature only works on M30 series and newer               |
|                                         | Firmware too old                 | Update to latest firmware version                        |
| **Slow startup after adjustment**       | Normal behavior with high upfreq | Lower upfreq speed for more stable startup               |
| **Efficiency loss**                     | Fast mode enabled                | Use Normal mode unless immediate response required       |
| **Cannot enable Fast Boot**             | Pre-M30 hardware                 | Feature not available on older models                    |

![Latest Version Example](assets/images/latest_version.jpg)

---

_Based on reverse-engineered firmware behavior and field testing. WhatsMiner doesn't officially document these features._
