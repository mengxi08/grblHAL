## grblHAL changelog

Build 20201103:

* Added data structures for spindle encoder/spindle sync to the core. Used by drivers supporting spindle sync.
* Updated spindle sync code for MSP432 and added spindle sync capability to iMXRT1060 and STM32F4xx drivers.  
__NOTE:__ Spindle sync support is still in alpha stage! The current code has only been tested with a simulator.
* Fixed bug that could lead to settings storage area fail to reinitialize properly when corrupted.
* Moved some symbols in preparation for adding $-settings for them.
* Improved rewind capability (on `M2`) of SD card plugin. Added `$` command for dumping SD card file contents to output stream.

Build 20201020:

__NOTE:__ Settings data format has been changed and settings will be reset to default on update. Backup and restore.

* Added support for STM32F446 based Nucleo-64 boards to STM32F4xx driver.
* Fix for regression that set laser mode as default - `$32=1`. Check that this is correct after restoring settings from backup. 
* Added new settings option `$341=4` for ignoring `M6` tool change command.
* Bug fix for `M61Q0` - returned error previously.
* Changed clearing of tool length offset reference on homing to be done only if relevant axis is/axes are homed.
* Fixed check for running startup scripts on homing to check that all axes configured for homing are actually homed.
* Changes to message display from `(MSG,..)` comments in gcode.  
`(MSG,..)` strings will be sent back to the sender in sync with gcode execution.
* Added `*` prefix to NVS storage type in `$I` report if buffered, e.g: `[NEWOPT:*EEPROM,ES,TC]`
* Increased heap allocation for nearly all drivers.  
Memory from heap is used for the NVS buffer and for temporary storage of `(MSG,..)` strings.  
* Added option for adding substates to the `Run` state in the real time report to `$11` setting \(bit 11\).  
If enabled `Run:2` will be reported when a probing motion is ongoing, this can be used by senders to provide a simple probe protection scheme.  
* Another refactoring of the settings subsystem, this time for handling plugin settings.
Plugins settings storage space is now dynamically allocated and handled locally by the plugin code, this allows user defined plugins to add settings too!
Ten setting codes are reserved for user defined plugins.
* Added template for [basic plugin with two settings](templates/README.md).
* HAL pointers refactored for code readability, many moved to separate structures for each subsystem. Added typedefs.
* Some minor bug fixes and other code readability improvements. Prepared for switch to 16-bit CRC for settings data validation.

Build 20200923:

* Added support for STM32F411 based Blackpill boards to STM32F4xx driver.
* Initial changes to ESP32 driver to allow compilation with PlatformIO, added my_machine.h for this. Note that my_machine.h is not used if compiling with idf.py.
* Added home position to `$#` ngc report, e.g. `[HOME,0.000,0.000,0.000:7]` - means all axes are homed. Position is reported in machine coordinates. `:7` in the example is an axis bitfield, the reported value is for which axes are homed: bit 0 is Z, 1 is X etc. `:0` = no axes homed.
* "Hardened" the new tool change functionality even more. Initial changes for multi-axis tool reference offset made.  
An empy message will now be sent when tool change is complete, this to clear any tool change related message in the sender.
* Added call to [weak](https://en.wikipedia.org/wiki/Weak_symbol) `my_plugin_init()` function at startup, name your [plugin](https://github.com/terjeio/grblHAL/tree/master/plugins) init function `void my_plugin_init (void)` and there is no need to change any grblHAL source files to bring it alive.  
Use this feature for your private plugin only, multiple public plugins using this name cannot coexist!
* Some changes to improve code readability and added strict check for `G59.x` gcodes.

Build 20200911:

* Core refactored for better support for non-volatile storage. Some HAL entry points renamed for readability and moved to a new data structure.
* Added plugin for axis odometers. This logs total distance traveled and machining time to EEPROM/FRAM.  
[FRAM](https://www.electronics-notes.com/articles/electronic_components/semiconductor-ic-memory/fram-ferroelectric-ram-memory.php) is recommended for storage as it is faster and can sustain a larger number of write cycles. FRAM chips are sold in packages that is pin compatible with EEPROM.  
__NOTE:__ Currently for review and for now only for the iMRXT1061 \(Teensy 4.x\) driver. It will _not_ be available in configurations that stores non-volatile data to flash.
* "Hardening" of new [manual tool change](https://github.com/terjeio/grblHAL/wiki/Manual,-semi-automatic-and-automatic-tool-change) functionality. 
* Improved auto squaring algorithm in core.
* Enhanced some plugins so they can coexist.

--- 

Build 20200830:

* Improved tool change functionality, a status report is forced on end of tool change to inform change is completed.
* Fix for iMRXT1062 homing failure when serial over USB is used.
* Standardized serial over USB `#define` macro across drivers to `USB_SERIAL_CDC`.

---

Build 20200818:

* Part two of large overhaul of configuration system.

Driver configuration has been moved from _driver.h_ to _my_machine.h_ and options has been made available to be defined as compiler defined symbols/macros with the `-D` compiler command line option. In order to allow compiler defined symbols to override _my_machine.h_ settings the symbol `OVERRIDE_MY_MACHINE` should always be added to the command line. See [compiling grblHAL](https://github.com/terjeio/grblHAL/wiki/Compiling-GrblHAL) for more information. Available driver options in _my_machine.h_ can now be listed from the driver ReadMe page.

---

Build 20200813:

* "Hardened" step pulse generation when delay is enabled. The delay will now only be added on direction changes.
The minimum possible delay is dependent on many factors and will be adjusted to match what the processor is capable of.
By default driver code is calibrated for a 2.5 &micro;s delay and may deviate a bit for other settings and configurations. The actual delay should be checked with an oscilloscope when high step rates are used.
__NOTE:__ A delay is only added if AMASS is disabled, when AMASS is enabled \(it is by default\) there is always a implicit delay on direction changes.

---

Build 20200811:

* Part one of large overhaul of configuration system.
This has been done to allow a global configuration file and/or setting compile time options as compiler symbols/macros with the `-D` compiler command line option.
* Networking \(cabled ethernet\), SD Card and I2C keypad plugin support added to [Teensy 4.x driver.](drivers/IMXRT1062) A Teensy 4.1 is required for networking.
* Tool number range changed from 8-bit \(0-255\) to 32-bit \(0-4294967294\). Note that if the optional tool table is enabled the max tool number is limited by number of entries in the tool table.
* M60 and corresponding pallet shuttle HAL entry point added. No driver support for this yet.

_Configuration system changes:_

Symbols \(#define macros\) that are unlikely to be changed or is used for conditional compilation has been moved from [grbl/config.h](grbl/config.h) to [grbl/grbl.h](grbl/grbl.h).
Symbols used for conditional compilation can be overridden by `-D` compiler command line options or by editing [grbl/config.h](grbl/config.h).

[grbl/defaults.h](grbl/defaults.h) is now used for default values for settings that can be changed at run-time with the `$$` command. __Important:__ grbl/defaults.h should only be changed when adding new settings.

[grbl/config.h](grbl/config.h) is now used for overriding default values in grbl/grbl.h or grbl/defaults.h. Out of the box all overridable symbols are commented out, uncomment and change as needed to override default definitions in grbl/grbl.h or grbl/defaults.h.

These changes are part of a long term plan to create a user friendly front end for configuring and building grblHAL.

---

Build 20200805:
* **Important:** settings has been changed again and settings will be restored to defaults after updating. Backup & restore!
* Added tool change handler for [manual tool change](https://github.com/terjeio/grblHAL/wiki/Manual,-semi-automatic-and-automatic-tool-change) on M6. Four different modes available, selectable with [`$341` setting](https://github.com/terjeio/grblHAL/wiki/Additional-or-extended-settings#manual-tool-change-settings).
* STMF4xx driver updated for optional I2C EEPROM plugin support.

---

Build 20200722:
* **Important:** settings version has been changed again and settings will be restored to defaults after updating. Backup & restore! 
* Changed step pulse width and delay settings from int to float and reduced minimum allowed value to 2 microseconds<sup>1</sup>. Useful for very high step rates.
* New plugin for [quadrature encoder input](https://github.com/terjeio/grblHAL/issues/73#issuecomment-659222664) for up to 5 encoders \(driver dependent\). Can be used to adjust overrides and has rudimentary support for MPG functionality. Work in progress and the iMXRT1062 \(Teensy 4\) driver is currently the only driver with low-level support for this (one encoder).
* New plugin for [ModBus VFD](https://github.com/terjeio/grblHAL/issues/68) spindle controllers. Untested and with limited driver support in this build.
* Added setting, `$340` for spindle at speed tolerance \(percent\). If spindle fails to reach speed within limits in 4 seconds alarm 14 will be raised. Set to 0 to disable. Availability driver dependent.
* All [new settings](https://github.com/terjeio/grblHAL/wiki/Additional-or-extended-settings) are now possibly to set independent of the [compatibility level](https://github.com/terjeio/grblHAL/wiki/Changes-from-grbl-1.1#workaround) except some settings that has flags added to them. The added flags will not be available at all compatibilty levels. A new command, `$+`, can be used to list all settings independent of compatibilty level.
* Internal changes to settings data in order to simplify automatic migration on changes. Automatic migration is on the roadmap.

<sup>1</sup> Note that several factors may affect the accuracy of these settings such as step output mode, number of axes defined and compiler optimization settings.
A new #define, `STEP_PULSE_LATENCY`, has been added to driver.h for those drivers that requires it so that fine tuning can be done. I may move this to a setting later.
In setups where very high step rates are used the actual step pulse width should be confirmed with an oscilloscope.

The step pulse delay has not been fine tuned yet as setting this to a value > 0 is not normally needed as there seems to be a implicit delay on direction changes when AMASS is enabled.

__Note:__ high step rates \(e.g. above 80 kHz\) cannot be achieved with the step pulse setting set to the default 10 microseconds. This has to be reduced so that both the on and off time is within specifications of the stepper driver. At 100 kHz the time available for a pulse \(on + off\) is 10 microseconds.

__Note:__ The SAMD21 \(MKRZERO\) driver needs updating in for it to allow short step pulses. My mistake was to use some Arduino code so that the Arduino pin numbers could be used for pin mappings. This adds around 500 ns per axis of overhead... To be fixed later.

For the curious: I have managed to achieve a 400 kHz step rate with the iMXRT1062 before everything breaks down, this with a command entered from MDI. I have not tested this with a running program, but I am pretty sure that a step rate at or above 200 kHz is sustainable.

---

2020/06/18: Added driver for STM32F4xx [Black Pill](https://www.cnx-software.com/2019/12/24/stm32-black-pill-board-features-stm32f4-cortex-m4-mcu-optional-spi-flash/), code modified by @shaise from the STM32F1xx driver. This is the first driver provided by someone else than me, thanks for that.

This driver is a candidate along with the IMXRT1062 \(Teensy 4.x\) driver to get spindle sync support. I have a [NucleoF411RE development board](https://www.st.com/en/evaluation-tools/nucleo-f411re.html) on order and will look into adding a pin mapping for that when it arrives. 

---


Build 20200603:
* **Important:** settings version has been changed and settings will be restored to defaults after updating. Backup & restore! 
* Optimizations for ring buffer handling in planner and step generator.
* New optional input signal for probe connected status, driver support will be added later to selected drivers.
* Automatic reporting of tool length offset \(`[TLO:...]`\) when changed.
* Support for [G5](http://www.linuxcnc.org/docs/2.5/html/gcode/gcode.html#sec:G5-Cubic-Spline) \(cubic spline\) added.
* `G43.x`, `G49` and `G92` added to parser state report.
* `G76` [threading cycle](https://hackaday.io/project/165248-mini-lathe-emco-compact-5-cnc-conversion) refactored.
* \(Re\)added `REPORT_PROBE_COORDINATES` and `TOOL_LENGTH_OFFSET_AXIS` [configuration](grbl/config.h) options, the latter available when `COMPATIBILITY_LEVEL` > 2.
* Improved backwards compatibility with vanilla grbl, e.g. G92 and tool offset\(s\) will be lost on a soft reset. Dependent on `COMPATIBILITY_LEVEL` setting.
* Board name added to `$I` report if provided by driver.
* [Grbl-Sim](https://github.com/grbl/grbl-sim) ported to grblHAL as a [driver](drivers/Simulator). Added telnet support++. Can be used to test senders. Note: currently only compiled/tested for Linux.
* Some minor bug fixes.

---

Build 20200503: Added configuration flag for manual homing. \(Re\)added compile time option `ENABLE_SAFETY_DOOR_INPUT_PIN` for [safety door switch](https://github.com/terjeio/grblHAL/blob/master/grbl/config.h), default is now disabled. Some bug fixes and "hardening" of code.

---

Added some [template code](./templates/README.md) to aid customizations such as driver support for M62 - M68 M-codes mentioned below.

---

Build 20191222: Added digital and analog output support to the core \(and HAL\) as per [linuxcnc specifications for M62 - M68](http://linuxcnc.org/docs/html/gcode/m-code.html#mcode:m62-m65), number of outputs available \(if any\) is driver dependent. Adding support for these M-commands makes it fairly easy to add driver code \(for up to 256 outputs\) as parsing and synchronization is taken care of by the core.

---

Build 20191215: Moved spindle RPM linearization to $-settings, option needs to be enabled in config.h - driver support required. Optimized EEPROM allocation handling. WebUI support for ESP32 driver improved.  

MSP432 driver enhanced for spindle linearization app in the pipeline \(for Windows only - needs input from spindle encoder\), more work done on closed loop spindle RPM control and spindle synchronized motion - still at experimental stage.

__NOTE:__ settings version number has been increased so settings will be reset to default after update, make a backup first!

---

Added [#define COMPATIBILITY_LEVEL to config.h](grbl/config.h) for backwards compatibility with Grbl v1.1 protocol definition, this for enabling the use of more GCode senders. Please raise an issue if your sender still does not behave well after setting this as the current implementation does not yet disable all extensions, notably [new $xx settings](doc/markdown/grblHAL%20extensions.md#settings).

G76 threading support added to grblHAL in combination with the [MSP432 driver](drivers/MSP432/README.md). Extensive testing is required before it can be regarded as safe.

**WARNING!** This is a potentially dangerous addition. Do NOT use if you do not understand the risks. A proper E-Stop is a must, it should cut power to the steppers and if possible engage any spindle brake. The implementation is based on the [linuxcnc specification](http://linuxcnc.org/docs/2.6/html/gcode/gcode.html#sec:G76-Threading-Canned). Please note that I am not a machinist so my interpretation and implementation may be wrong!

G76 availablity requires a spindle encoder with index pulse, grblHAL configured to [lathe mode](doc/markdown/settings.md#opmode) and tuning of the spindle sync PID loop.  
__NOTE:__ Feed hold is delayed until spindle synced cut is complete, spindle RPM overrides and CSS mode disabled through the whole cycle. 
