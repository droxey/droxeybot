# droxeybot

[**@droxey**](https://github.com/droxey)'s customized Ender 3 build featuring SKR Mini E3 V2.0 / Hemera / Filament Detection / BLTouch / UPS / Power Relay.

| Component                   | Part                                                                         | Repo                                                                                                                                              |
|-----------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| **Motherboard**             | **BTT SKR MINI E3 V2.0**                                                     | [BIGTREETECH-SKR-mini-E3](https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3/tree/master/firmware/V2.0/Marlin-2.0.x-SKR-mini-E3-V2.0)         |
| **Extruder**                | **[Hemera Direct Drive](https://e3d-online.com/e3d-hemera-175-kit)**         | -                                                                                                                                                 |
| **Power Supply**            | **MeanWell LRS-350-24U**                                                     | -                                                                                                                                                 |
| **Filament Sensor**         | **BTT Smart Filament Detection**                                             | [smart-filament-detection-module](https://github.com/bigtreetech/smart-filament-detection-module)                                                 |
| **Relay**                   | **BTT Relay v1.2**                                                           | [BIGTREETECH-Relay-V1.2](https://github.com/bigtreetech/BIGTREETECH-Relay-V1.2/tree/master/BIGTREETECH%20Relay%20V1.2/BIGTREETECH%20Relay%20V1.2) |
| **UPS**                     | **BTT UPS 24V V1.0**                                                         | [BIGTREETECH-MINI-UPS-V2.0](https://github.com/bigtreetech/BIGTREETECH-MINI-UPS-V2.0/tree/master/BTT%20UPS%2024V%20V1.0)                          |
| **Leveling /<br>Z Endstop** | **BLTouch v3.1**                                                             | -                                                                                                                                                 |
| **Dual Z Axis**             | **[ExoSlide XZ Kit](https://www.exoslide.com/products/kits/ender3-XZ)**      | -                                                                                                                                                 |
| **Upgraded Y Axis**         | **[ExoSlide Y Bed Kit](https://www.exoslide.com/products/kits/ender3-Ybed)** | -                                                                                                                                                 |
## Configure Firmware

```bash
$ git clone git@github.com:MarlinFirmware/Marlin.git
$ cd Marlin
$ git checkout bugfix-2.0.x
$ curl -o 1_UpdateFirmware.patch https://raw.githubusercontent.com/linsomniac/MarlinSKRMiniE3v2.0Files/master/essential_changes.patch
$ curl -o 2_BLTouch.patch https://raw.githubusercontent.com/linsomniac/MarlinSKRMiniE3v2.0Files/master/bltouch.patch
$ patch -p1 <1_UpdateFirmware.patch
$ patch -p1 <2_BLTouch.patch
```

## Connect Hardware & Add Features

### Step Daemon

> Step Daemon (`stepd`) is an external planner for 3d printers that utilizes Marlin compatible firmware to allow direct step processing by an external computer and enables the use of complex pre-processing.

Set it up on your Raspberry Pi by following the instructions in the project's [README](https://github.com/colinrgodsey/step-daemon).

### Jumpers

1. Remove the jumper from `Z_DIA3` to use the BLTouch as the Z endstop.
2. Remove the jumper above `Z_PROBE` to enable the onboard 5v power supply.

### BLTouch

<img src="https://droxey.com/statics/img/bltouch.png" style="zoom: 25%;" >

Connect the BLTouch to `Z_PROBE` on the motherboard.

### UPS

<img src="https://droxey.com/statics/img/ups.jpg" alt="img" style="zoom:33%;" />

Connect the UPS module to the `PWR_DE` pin on the motherboard.

Open `Configuration_adv.h` and modify the following settings to match the following snippet:

```c
 #define POWER_LOSS_RECOVERY
 #if ENABLED(POWER_LOSS_RECOVERY)
   #define PLR_ENABLED_DEFAULT   false // Power Loss Recovery enabled by default. (Set with 'M413 Sn' & M500)
   #define BACKUP_POWER_SUPPLY       // Backup power / UPS to move the steppers on power loss
   #define POWER_LOSS_ZRAISE       10 // (mm) Z axis raise on resume (on power loss with UPS)
   #define POWER_LOSS_PIN         PC12 // Pin to detect power loss. Set to -1 to disable default pin on boards without module.
   #define POWER_LOSS_STATE       HIGH // State of pin indicating power loss
   #define POWER_LOSS_PULL           // Set pullup / pulldown as appropriate
   #define POWER_LOSS_PURGE_LEN   20 // (mm) Length of filament to purge on resume
   #define POWER_LOSS_RETRACT_LEN 10 // (mm) Length of filament to retract on fail. Requires backup power.

   // Without a POWER_LOSS_PIN the following option helps reduce wear on the SD card,
   // especially with "vase mode" printing. Set too high and vases cannot be continued.
   #define POWER_LOSS_MIN_Z_CHANGE 0.05 // (mm) Minimum Z change before saving power-loss data
 #endif
```

### Relay

<img src="https://droxey.com/statics/img/relay.jpg" alt="img" style="zoom:33%;" />

Connect the relay to your power supply according to the diagram. Connect the relay to the `PS_ON` pin on your motherboard.

Open `Configuration.h` and make the following modifications to the firmware:

```c
/**
 * Power Supply Control
 *
 * Enable and connect the power supply to the PS_ON_PIN.
 * Specify whether the power supply is active HIGH or active LOW.
 */
#define PSU_CONTROL
#define PSU_NAME "Power Supply"

#if ENABLED(PSU_CONTROL)
  #define PSU_ACTIVE_HIGH true     // Set 'false' for ATX, 'true' for X-Box

  //#define PSU_DEFAULT_OFF         // Keep power off until enabled directly with M80
  //#define PSU_POWERUP_DELAY 100   // (ms) Delay for the PSU to warm up to full power

  // #define AUTO_POWER_CONTROL      // Enable automatic control of the PS_ON pin
  #if ENABLED(AUTO_POWER_CONTROL)
    #define AUTO_POWER_FANS         // Turn on PSU if fans need power
    #define AUTO_POWER_E_FANS
    #define AUTO_POWER_CONTROLLERFAN
    #define AUTO_POWER_CHAMBER_FAN
    //#define AUTO_POWER_E_TEMP        50 // (°C) Turn on PSU over this temperature
    //#define AUTO_POWER_CHAMBER_TEMP  30 // (°C) Turn on PSU over this temperature
    #define POWER_TIMEOUT 30
  #endif
#endif
```

Finally, in your slicing software, add `M81` to the bottom of your end script.

### Smart Filament Detection Module

1. To configure the **Smart Filament Runout Sensor**, open `Configuration.h` and uncomment `#define FILAMENT_RUNOUT_SENSOR`. Then, **ensure your settings match the following**:

   ```c
   /**
    * Filament Runout Sensors
    * Mechanical or opto endstops are used to check for the presence of filament.
    *
    * RAMPS-based boards use SERVO3_PIN for the first runout sensor.
    * For other boards you may need to define FIL_RUNOUT_PIN, FIL_RUNOUT2_PIN, etc.
    * By default the firmware assumes HIGH=FILAMENT PRESENT.
    */
   #define FILAMENT_RUNOUT_SENSOR
   #define FIL_RUNOUT_PIN PC15 // Filament detection pin for BTT SKR Mini E3 v2.0
   #if ENABLED(FILAMENT_RUNOUT_SENSOR)
       #define NUM_RUNOUT_SENSORS 1 // Number of sensors, up to one per extruder. Define a FIL_RUNOUT#_PIN for each.
       #if ENABLED(INVERT_FS_LOGIC)
           #define FIL_RUNOUT_INVERTING true // Trigger's alternative as soon as invert filamentsensor logic is activated
       #else
           #define FIL_RUNOUT_INVERTING false // Logic inverting is automatically taken care in section 13
       #endif
       // Set to true to invert the logic of the sensor.
       #if ENABLED(INVERTL_PINPULLUP_LOGIC)
           #define FIL_RUNOUT_PULLDOWN // Use internal pulldown for filament runout pins.
       #else
           #define FIL_RUNOUT_PULLUP // Use internal pullup for filament runout pins.
       #endif

       // Set one or more commands to execute on filament runout.
       // (After 'M412 H' Marlin will ask the host to handle the process.)
       #define FILAMENT_RUNOUT_SCRIPT "M600"

       // After a runout is detected, continue printing this length of filament
       // before executing the runout script. Useful for a sensor at the end of
       // a feed tube. Requires 4 bytes SRAM per sensor, plus 4 bytes overhead.
       #define FILAMENT_RUNOUT_DISTANCE_MM 7

       #ifdef FILAMENT_RUNOUT_DISTANCE_MM
       // Enable this option to use an encoder disc that toggles the runout pin
       // as the filament moves. (Be sure to set FILAMENT_RUNOUT_DISTANCE_MM
       // large enough to avoid false positives.)
           #define FILAMENT_MOTION_SENSOR
       #endif
   #endif
   ```

2. In `Configuration_adv.h`, make the following changes:

   1. To enable runout detection on serial LCDs, uncomment `#define M114_DETAIL`. This allows runout detection to function in Marlin Mode.
   2. To add filament change support, uncomment `#define ADVANCED_PAUSE_FEATURE`. This feature also enables nozzle park when paused without further firmware modifications.

## Resources & Credits

- Initial configuration based on [u/qwewer1](https://www.reddit.com/user/qwewer1/) post on Reddit, [Marlin 2.0.x guide, SKR Mini E3 v2.0, Ender 3](https://www.reddit.com/r/ender3/comments/h8y1ia/marlin_20x_guide_skr_mini_e3_v20_ender_3/) on 06/14/2020.
- Shoutout to [@linsomniac](https://github.com/linsomniac)'s repository, [linsomniac/MarlinSKRMiniE3v2.0Files](https://github.com/linsomniac/MarlinSKRMiniE3v2.0Files). These patches are used to augment vanilla Marlin to support the SKR Mini E3 v2.0 motherboard.
- [Step Daemon for Marlin](https://github.com/colinrgodsey/step-daemon).
