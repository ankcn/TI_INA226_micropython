# TI_INA226_micropython

This library provides support for the TI INA226 power measurement IC with micropython firmware.
Datasheet and other information on the IC: https://www.ti.com/product/INA226
#  
Based on https://github.com/elschopi/TI_INA226_micropython which is derived from https://github.com/robert-hh/INA219


# Basics

To use the device, it has to be configured at startup. In it's default configuration, the calibration register is not set and 
thus the current and power cannot be directly read out.</br>
By default, this library configures the device to a maximum current of 0.75 A and 36V bus voltage. Resistance of the shunt is assumed as 0.1 Ohm.


# Usage information

It is necessary to set the operating mode of the IC and to tell the parameters of the shunt. Both actions are performed by calling the calibrate function. After that, you can read the values ​​of current, voltage and power. See examples in the next section.  
If you have a general purpose resistor as a shunt, then you just pass its resistance in mOhm. Otherwise, for dedicated high current shunt resistor, specify it nominal current in A. Typicaly many shunts have a 75 mV dropout voltage at nominal current and this is default value in this library. But you may pass a different value.  
If both of two optional parameters - maximum expected current and shunt resistance are passed, then the resistance will be ignored.

Default configuration:
- Averaging mode: 512 samples
- Bus voltage conversion time: 588 us
- Shunt voltage conversion time: 588 us
- Operating mode: Shunt and Bus voltage, continuous

High averaging samples number and conversion times takes more accurate results.  
Predefined configuration constants can be found in the library.


# Code examples
Simple case with all arguments leaved default. Suitable for module with 0.1 Ohm resistor as a shunt.

```python
import ina226
from machine import Pin, I2C

i2c = I2C(scl=Pin(2), sda=Pin(0))
ina = ina226.INA226(i2c)
ina.calibrate()

print(ina.bus_voltage)
print(ina.shunt_voltage)
print(ina.current)
print(ina.power)
```

If you have a 0.01 Ohm shunt resistor:

```python
ina.calibrate(r_shunt=10)
```

Custom operating mode and 10 A shunt:

```python
ina.calibrate(
	ina226.CONFIG_AVGMODE_256SAMPLES |
	ina226.CONFIG_VBUSCT_2116us |
	ina226.CONFIG_MODE_SANDBVOLT_CONTINUOUS |
	ina226.CONFIG_CONST_BITS |
	ina226.CONFIG_VSHUNTCT_2116us,
	10)
```

Utilizing almost full shunt voltage range of the INA226 (81.92mV) and potentialy slight overloading 10 A shunt:

```python
ina.calibrate(max_current=10.8, v_shunt=81)
```
