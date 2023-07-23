# Detailed Configuration Guide

## ![#f03c15](/doc/f03c15.png) ![#c5f015](/doc/c5f015.png) ![#1589F0](/doc/1589F0.png) mmu_parameters.cfg

The first section specifies the type of MMU and is used by Happy Hare to adjust options. It is documented in the main [README.md](https://github.com/moggieuk/Happy-Hare#1-important-mmu-vendor--version-specification).

```yml
[mmu]
#
# The vendor and version config is important to define the capabiliies of the MMU
#
# ERCF
# 1.1 original design, add "s" suffix for Sprigy, "b" for Binky, "t" for Triple-Decky
#     e.g. "1.1sb" for v1.1 with Spriny mod and Binky encoder
# 2.0 new community edition ERCF
#
# Tradrack
#  - Comming soon
#
# Prusa
#  - Comming soon
#
mmu_vendor: ERCF			# MMU family
mmu_version: 1.1			# MMU hardware version number (add mod suffix documented above)
mmu_num_gates: 9			# Number of selector gates

The servo configuration allos for up to three positions but some designs (e.g. ERCF v1.1) only require `up`/`down`.  If `move` is not used then comment it out or set it to the same value as `up`.  The servo duraction is the lemght of PWM burst.  Most digital servos only require a short 0.1 second or so but slower analog servos may require longer (0.4 - 0.5s).  Be very careful if you use the `servo_active_down` option because it will can strain your electronics.

# Servo configuration  -----------------------------------------------------------------------------------------------------
#
# Angle of the servo in three named positions:
#   up   = tool is selected and filament is allowed to freely move through gate
#   down = to grip filament
#   move = ready the servo for selector move (optional - defaults to up)
#
# Note that leaving the servo active when down can stress the electronics and is not recommended with EASY-BRD or ERB board
# unless the 5v power supply has been improved and it is not necessary with standard ERCF build.
# Make sure your hardware is suitable for the job!
#
servo_up_angle: 125			# Default: MG90S servo: Up~30    ; SAVOX SH0255MG: Up~140
servo_down_angle: 45			# Default: MG90S servo: Down~140 ; SAVOX SH0255MG: Down~30
servo_move_angle: 110			# Optional angle used when selector is moved (defaults to up position)
servo_duration: 0.2			# Duration of PWM burst sent to servo (automatically turns off)
servo_active_down: 0			# CAUTION: 1=Force servo to stay active when down, 0=Release after movement
```

Logging controls control the verbosity level of logging to console and separate `mmu.log` file as well and fun visual filament position and various status messages - it really is unessessary to have verbose logging to the console so defaults are recommended.

```yml
# Logging ------------------------------------------------------------------------------------------------------------------
#
# log_level & logfile_level can be set to one of (0 = essential, 1 = info, 2 = debug, 3 = trace, 4 = developer)
# Generally you can keep console logging to a minimal whilst still sending debug output to the mmu.log file
# Increasing the console log level is only really useful during initial setup to save having to constantly open the log file
#
log_level: 1
log_file_level: 3			# Can also be set to -1 to disable log file completely
log_statistics: 1 			# 1 to log statistics on every toolchange (default), 0 to disable (but still recorded)
log_visual: 2				# 1 log visual representation of filament, 2 compact form (default) , 0 disable
log_startup_status: 1			# Whether to log tool to gate status on startup, 1 = summary (default), 2 = full, 0 = disable

All Happy Hare speeds can be configured in this section.  Most are self-explanatory and are separated into gear stepper speeds, speeds inside of the extruder (either just extruder motor or when synced with gear stepper) and selector movement.

# Movement speeds ----------------------------------------------------------------------------------------------------------
#
# Long moves are faster than the small ones and used for the bulk of the bowden movement. Note that you can set two fast load
# speeds depending on whether MMU thinks it is pulling from the buffer or from the spool. It is often helpful to use a lower
# speed when pulling from the spool because more force is required to overcome friction and this prevents loosing steps.
# 100mm/s should be "quiet" with the NEMA14 motor or a NEMA17 pancake, but you can go lower for really low noise
# NOTE: Encoder cannot keep up much above 250mm/s so make sure `apply_bowden_correction` is off at very high speeds!
#
gear_from_buffer_speed: 160		# mm/s Conservative value is 100mm/s, Max around 350mm/s
gear_from_spool_speed: 60		# mm/s Use (lower) speed when loading from a gate for the first time (i.e. pulling from spool)
gear_short_move_speed: 60		# mm/s Conservative value is 35mm/s. Max around 100mm/s
gear_homing_speed: 50			# mm/s Speed of gear stepper only homing moves (e.g. extruder homing)

# Speeds of extruder movement. The 'sync' speeds will be used when gear and extruder steppers are moving in sync
extruder_load_speed: 15			# mm/s speed of load move inside extruder from homing position to meltzone
extruder_unload_speed: 20		# mm/s speed of unload moves inside of extruder (very initial move from meltzone is 50% of this)
extruder_sync_load_speed: 20		# mm/s speed of synchronized extruder load moves
extruder_sync_unload_speed: 25		# mm/s speed of synchronized extruder unload moves
extruder_homing_speed: 20		# mm/s speed of extruder only homing moves (e.g. to toolhead sensor)

# Selector movement speeds
selector_move_speed: 200        	# mm/s speed of selector movement (not touch)
selector_homing_speed: 60       	# mm/s speed of initial selector homing move (not touch)
selector_touch_speed: 80		# mm/s speed of all touch selector moves (if stallguard configured)
enable_selector_touch: 0		# If selector touch operation is possible this can be used to disable it 1=enabled, 0=disabled
```

For more information on the bowden correct move, read about the loading sequence [here](https://github.com/moggieuk/Happy-Hare#---filament-loading-and-unloading-sequences).  The `bowden_num_moves` allows a long move to be broken into separate moves.  Only increase this if Klipper throws errors with very long moves - setting it higher than `1` will long down the loading process.

```yml
# Bowden tube loading/unloading --------------------------------------------------------------------------------------------
#
# In addition to different bowden loading speeds for buffer and non-buffered filament it is possible to detect missed steps
# caused by "jerking" on a heavy spool. If bowden correction is enabled the driver with "believe" the encoder reading and
# make correction moves to bring the filament to within the 'load_bowden_tolerance' of the end of bowden position
# (this does require a reliable encoder and is not recommended for very high speed loading >200mm/s)
#
bowden_apply_correction: 0		# 1 to enable, 0 disabled (default)
bowden_load_tolerance: 15.0		# How close in mm the correction moves will attempt to get to target
bowden_num_moves: 1			# Number of separate fast moves to make when loading or unloading bowden (>1 if you have TTC errors)
```

This section controls the optional extruder homing step. The `extruder_homing_endstop` is either a real endstop name or the string "collision" which causes Happy Hare to "feel" for the extruder entrance.  If other options dictate this homing step it will automatically be performed, however it is possible to force it even when not strickly needed by setting the `extruder_force_homing: 1`.


Logging controls control the verbosity level of logging to console and separate `mmu.log` file as well and fun visual filament position and various status messages - it really is unessessary to have verbose logging to the console so defaults are recommended.

```yml
# Logging ------------------------------------------------------------------------------------------------------------------
#
# log_level & logfile_level can be set to one of (0 = essential, 1 = info, 2 = debug, 3 = trace, 4 = developer)
# Generally you can keep console logging to a minimal whilst still sending debug output to the mmu.log file
# Increasing the console log level is only really useful during initial setup to save having to constantly open the log file
#
log_level: 1
log_file_level: 3			# Can also be set to -1 to disable log file completely
log_statistics: 1 			# 1 to log statistics on every toolchange (default), 0 to disable (but still recorded)
log_visual: 2				# 1 log visual representation of filament, 2 compact form (default) , 0 disable
log_startup_status: 1			# Whether to log tool to gate status on startup, 1 = summary (default), 2 = full, 0 = disable

All Happy Hare speeds can be configured in this section.  Most are self-explanatory and are separated into gear stepper speeds, speeds inside of the extruder (either just extruder motor or when synced with gear stepper) and selector movement.

# Movement speeds ----------------------------------------------------------------------------------------------------------
#
# Long moves are faster than the small ones and used for the bulk of the bowden movement. Note that you can set two fast load
# speeds depending on whether MMU thinks it is pulling from the buffer or from the spool. It is often helpful to use a lower
# speed when pulling from the spool because more force is required to overcome friction and this prevents loosing steps.
# 100mm/s should be "quiet" with the NEMA14 motor or a NEMA17 pancake, but you can go lower for really low noise
# NOTE: Encoder cannot keep up much above 250mm/s so make sure `apply_bowden_correction` is off at very high speeds!
#
gear_from_buffer_speed: 160		# mm/s Conservative value is 100mm/s, Max around 350mm/s
gear_from_spool_speed: 60		# mm/s Use (lower) speed when loading from a gate for the first time (i.e. pulling from spool)
gear_short_move_speed: 60		# mm/s Conservative value is 35mm/s. Max around 100mm/s
gear_homing_speed: 50			# mm/s Speed of gear stepper only homing moves (e.g. extruder homing)

# Speeds of extruder movement. The 'sync' speeds will be used when gear and extruder steppers are moving in sync
extruder_load_speed: 15			# mm/s speed of load move inside extruder from homing position to meltzone
extruder_unload_speed: 20		# mm/s speed of unload moves inside of extruder (very initial move from meltzone is 50% of this)
extruder_sync_load_speed: 20		# mm/s speed of synchronized extruder load moves
extruder_sync_unload_speed: 25		# mm/s speed of synchronized extruder unload moves
extruder_homing_speed: 20		# mm/s speed of extruder only homing moves (e.g. to toolhead sensor)

# Selector movement speeds
selector_move_speed: 200        	# mm/s speed of selector movement (not touch)
selector_homing_speed: 60       	# mm/s speed of initial selector homing move (not touch)
selector_touch_speed: 80		# mm/s speed of all touch selector moves (if stallguard configured)
enable_selector_touch: 0		# If selector touch operation is possible this can be used to disable it 1=enabled, 0=disabled
```

For more information on the bowden correct move, read about the loading sequence [here](https://github.com/moggieuk/Happy-Hare#---filament-loading-and-unloading-sequences).  The `bowden_num_moves` allows a long move to be broken into separate moves.  Only increase this if Klipper throws errors with very long moves - setting it higher than `1` will long down the loading process.


Logging controls control the verbosity level of logging to console and separate `mmu.log` file as well and fun visual filament position and various status messages - it really is unessessary to have verbose logging to the console so defaults are recommended.

```yml
# Logging ------------------------------------------------------------------------------------------------------------------
#
# log_level & logfile_level can be set to one of (0 = essential, 1 = info, 2 = debug, 3 = trace, 4 = developer)
# Generally you can keep console logging to a minimal whilst still sending debug output to the mmu.log file
# Increasing the console log level is only really useful during initial setup to save having to constantly open the log file
#
log_level: 1
log_file_level: 3			# Can also be set to -1 to disable log file completely
log_statistics: 1 			# 1 to log statistics on every toolchange (default), 0 to disable (but still recorded)
log_visual: 2				# 1 log visual representation of filament, 2 compact form (default) , 0 disable
log_startup_status: 1			# Whether to log tool to gate status on startup, 1 = summary (default), 2 = full, 0 = disable
```

All Happy Hare speeds can be configured in this section.  Most are self-explanatory and are separated into gear stepper speeds, speeds inside of the extruder (either just extruder motor or when synced with gear stepper) and selector movement.

