<a id="library"></a>
# gbjBH1750FVI
Library for digital ambient light sensor BH1750FVI with two-wire (I2C) bus interface usually on board GY-302. Library enables changing address of the sensor dynamically, if ADDR pin of the sensor is connected to a microcontroller pin and changed programmatically accordingly as well as changing measurement mode.

#### Particle - Photon hardware configuration
- Connect microcontroller's pin `D0` to sensor's pin **SDA** (Serial Data).
- Connect microcontroller's pin `D1` to sensor's pin **SCL** (Serial Clock).
- Connect microcontroller's pin `D2` to sensor's pin **ADDR** (Addressing) in order to set its address programmatically.

#### AVR - Arduino UNO hardware configuration
- Connect microcontroller's pin `A4` to sensor's pin **SDA** (Serial Data).
- Connect microcontroller's pin `A5` to sensor's pin **SCL** (Serial Clock).
- Connect microcontroller's pin `D2` to sensor's pin **ADDR** (Addressing) in order to set its address programmatically.

#### Espressif - ESP8266, ESP32 default hardware configuration
- Connect microcontroller's pin `D2` to sensor's pin **SDA** (Serial Data).
- Connect microcontroller's pin `D1` to sensor's pin **SCL** (Serial Clock).
- Connect microcontroller's pin `D0` to sensor's pin **ADDR** (Addressing) in order to set its address programmatically.


<a id="credit"></a>
## Credit
Library has been inspired by the *Christopher Laws*'s library *BH1750-master version 1.1.3* but totally redesigned and rewritten.


<a id="dependency"></a>
## Dependency

#### Particle platform
- **Particle.h**: Includes alternative (C++) data type definitions.

#### Arduino platform
- **Arduino.h**: Main include file for the Arduino SDK version greater or equal to 100.
- **WProgram.h**: Main include file for the Arduino SDK version less than 100.
- **inttypes.h**: Integer type conversions. This header file includes the exact-width integer definitions and extends them with additional facilities provided by the implementation.
- **TwoWire**: I2C system library loaded from the file *Wire.h*.

#### Custom Libraries
- **gbjTwoWire**: I2C custom library loaded from the file *gbj_twowire.h*. The library [gbjBH1750FVI](#library) inherits common bus functionality from this library.


<a id="constants"></a>
## Constants
- **GBJ\_BH1750FVI\_VERSION**: Name and semantic version of the library.
- **GBJ\_BH1750FVI\_ADDRESS\_L**: Sensor address (0x23) for low ADDR pin state with voltage <= 0.3 Vcc.
- **GBJ\_BH1750FVI\_ADDRESS\_H**: Sensor address (0x5C) for high ADDR pin state with voltage >= 0.7 Vcc.
- **GBJ\_BH1750FVI\_CONTINUOUS\_HIGH**: Start measurement at 1 lx resolution. Measurement time is typically 120 ms.
- **GBJ\_BH1750FVI\_CONTINUOUS\_HIGH2**: Start measurement at 0.5 lx resolution. Measurement time is typically 120 ms.
- **GBJ\_BH1750FVI\_CONTINUOUS\_LOW**: Start measurement at 4 lx resolution. Measurement time is typically 16 ms.
- **GBJ\_BH1750FVI\_ONETIME\_HIGH**: Start measurement at 1 lx resolution. Measurement time is typically 120 ms. The sensor is automatically set to Power Down mode after measurement.
- **GBJ\_BH1750FVI\_ONETIME\_HIGH2**: Start measurement at 0.5 lx resolution. Measurement time is typically 120 ms. The sensor is automatically set to Power Down mode after measurement.
- **GBJ\_BH1750FVI\_ONETIME\_LOW**: Start measurement at 4 lx resolution. Measurement time is typically 16 ms. The sensor is automatically set to Power Down mode after measurement.
- **GBJ\_BH1750FVI\_VALUE\_BAD**: Virtual value for a bad measurement, which can never occur at correct processing.


<a id="errors"></a>
#### Error codes
- **GBJ\_BH1750FVI\_ERR\_MODE**: Bad measurement mode

Other error codes as well as result codes provides the parent library [gbjTwoWire](#dependency) including other macro constants.
Remaining constants are listed in the library include file. They are used mostly internally as function codes of the sensor.


<a id="interface"></a>
## Interface
The library does not need special constructor and destructor, so that the inherited ones are good enough and there is no need to define them in the library, just use it:
```cpp
  gbj_bh1750fvi Light = gbj_bh1750fvi();
```

#### Main
- [begin()](#begin)
- [powerOn()](#power)
- [powerDown()](#power)
- [reset()](#reset)
- [measureLight()](#measureLight)

#### Setters
- [setAddress()](#setAddress)
- [setMode()](#setMode)

#### Getters
- [getMode()](#getMode)
- [getLight()](#getLight)
- [getLightMSB()](#getLightMSB)
- [getLightLSB()](#getLightLSB)

Other possible setters and getters are described at parent library [gbjTwoWire](#dependency).

<a id="begin"></a>
## begin()
#### Description
The method takes, sanitizes, and stores parameters of the sensor to a class instance object and initiates two-wire bus.
- You can use [setters](#interface) later in order to change sensor's parameters.
- Setting sensor parameters in this method located in the setup section of an application instead of constructor enables to set the sensor's address according to the ADDR pin, if it is wired in that way.

#### Syntax
    uint8_t begin(uint8_t address, uint8_t mode, boolean busStop);

#### Parameters
<a id="prm_address"></a>
- **address**: One of two possible 7 bit addresses of the sensor or state of the ADDR pin.
  - *Valid values*: non-negative integer [GBJ\_BH1750FVI\_ADDRESS\_L](#constants) or [GBJ\_BH1750FVI\_ADDRESS\_H](#constants) or pin states **HIGH** or **LOW**.
  - *Default value*: [GBJ\_BH1750FVI\_ADDRESS\_DEF](#constants)
	  - The default values is set to address corresponding to not wired ADDR pin, which is equivalent to the connection to the ground.

<a id="prm_mode"></a>
- **mode**: Measurement mode from possible listed ones.
  - *Valid values*: [GBJ\_BH1750FVI\_CONTINUOUS\_HIGH](#constants) ~ [GBJ\_BH1750FVI\_ONETIME\_LOW](#constants)
  - *Default value*: [GBJ\_BH1750FVI\_MODE\_DEF](#constants)

<a id="prm_busStop"></a>
- **busStop**: Logical flag about releasing bus after end of transmission.
  - *Valid values*: true, false
	  - **true**: Releases the bus after data transmission and enables other master devices to control the bus.
    - **false**: Keeps connection to the bus and enables begin further data transmission immediately.
  - *Default value*: true

#### Returns
Some of result codes defined by [Constants](#constants) including ones from the parent library.

#### Example
The method has all arguments defaulted and calling without any parameters is equivalent to the calling with all arguments set by corresponding constant with default value:

``` cpp
gbj_bh1750fvi Light = gbj_bh1750fvi();
setup()
{
    Light.begin();  // It is equivalent to
    Light.begin(GBJ_BH1750FVI_ADDRESS_DEF, GBJ_BH1750FVI_MODE_DEF, true);
}
```

If some argument after some defaulted arguments should have a specific value, use corresponding constants in place of those defaulted arguments, e.g.,

``` cpp
Light.begin(GBJ_BH1750FVI_ADDRESS_DEF, GBJ_BH1750FVI_ONETIME_LOW);      // Specific measurement mode
Light.begin(GBJ_BH1750FVI_ADDRESS_DEF, GBJ_BH1750FVI_MODE_DEF, false);  // Keeping bus connected after each data transmission
```

Typical usage is just with default values without any arguments.
Specific values of arguments can be set by corresponding [setters](#interface).

#### See also
[setAddress()](#setAddress)

[setMode()](#setMode)

[Back to interface](#interface)


<a id="power"></a>
## powerOn(), powerDown()
#### Description
The particular method either activates (wakes up) or deactivates (sleeps) a sensor.
In active state a sensor waits for the measurement command.
In sleeping state a sensor has minimal power consumption.

#### Syntax
    uint8_t powerOn();
    uint8_t powerDown();

#### Parameters
None

#### Returns
Some of result or [error codes](#errors).

#### See also
[reset()](#reset)

[Back to interface](#interface)


<a id="reset"></a>
## reset()
#### Description
The method resets the illuminance data register of the sensor and removes previous measurement result.

#### Syntax
    uint8_t reset();

#### Parameters
None

#### Returns
Some of result or [error codes](#errors).

#### See also
[measureLight()](#measureLight)

[Back to interface](#interface)


<a id="measureLight"></a>
## measureLight()
#### Description
The method measures the ambient light intensity in lux at 16-bit resolution with accuracy determined by measurement mode while uses the parameters set by either the method [begin()](#begin) or setters. The method stores the measured value in the class instance object for repeating retrieval without need to measure again.
- Maximal light intensity value is 54612 lux, because of maximal unsigned integer value 65535 in the internal registers of the sensor and divided by measurement coefficient 1.2, i.e.,
  `65535 / 1.2 = 54612.5` and truncating decimal part.

#### Syntax
    uint16_t measureLight();

#### Parameters
None

#### Returns
Current ambient light intensity in the range 0 ~ 54612 lux or value [GBJ\_BH1750FVI\_VALUE\_BAD](#constants) signaling erroneous measurement. The corresponding [error code](#errors) is set as well.

#### See also
[getLight()](#getLight)

[getLightMSB()](#getLightMSB)

[getLightLSB()](#getLightLSB)

[Back to interface](#interface)


<a id="setAddress"></a>
## setAddress()
#### Description
The method overrides the method of the parent class by transforming and sanitizing input value, which can be a state of the ADDR pin.
- If the input address is an address value, it can be just one of valid addresses [GBJ\_BH1750FVI\_ADDRESS\_L](#constants) or [GBJ\_BH1750FVI\_ADDRESS\_H](#constants).
- If the input address is the ADDR pin state, it can be either HIGH or LOW.
- In fact, the ADDR pin is not aimed to utilize dynamic changing the device address, but to enable to use two sensors on one two-wire bus.

#### Syntax
    uint8_t setAddress(uint8_t address);

#### Parameters
- **address**: The address value or the ADDR pin state, so the same range as the argument [address](#prm_address) in the method [begin()](#begin).

#### Returns
Some of result or [error codes](#errors).

#### Example
```cpp
Light.setAddress(digitalRead(PIN_BH1750FVI_ADDR));
Light.setAddress(GBJ_BH1750FVI_ADDRESS_H);
```

#### See also
[begin()](#begin)

[Back to interface](#interface)


<a id="setMode"></a>
## setMode()
#### Description
The method sets the measurement mode of the sensor. It should be one of defined by [Constants](#constants).

#### Syntax
    uint8_t setMode(uint8_t mode);

#### Parameters
- **mode**: The same as the argument [mode](#prm_mode) in the method [begin()](#begin).

#### Returns
Some of result or [error codes](#errors).

#### See also
[begin()](#begin)

[getMode()](#getMode)

[Back to interface](#interface)


<a id="getMode"></a>
## getMode()
#### Description
The method returns the current measurement mode of the sensor stored in the class instance object.

#### Syntax
    uint8_t getMode();

#### Parameters
None

#### Returns
Current measurement mode of the sensor.

#### See also
[setMode()](#setMode)

[Back to interface](#interface)


<a id="getLight"></a>
## getLight()
#### Description
The method retrieves recently measured light intensity without need to measure again, e.g., for repeating usage without local variable.

#### Syntax
    uint16_t getLight();

#### Parameters
None

#### Returns
Recently measured light intensity in lux.

#### See also
[measureLight()](#measureLight)

[Back to interface](#interface)


<a id="getLightMSB"></a>
## getLightMSB()
#### Description
The method retrieves most significant byte of the raw sensor value from recent measurement. It is suitable for testing purposes or at very restrictive data transfers, e.g., over the air.

#### Syntax
    uint8_t getLightMSB();

#### Parameters
None

#### Returns
High byte of the recently measured sensor value.

#### See also
[getLight()](#getLight)

[Back to interface](#interface)


<a id="getLightLSB"></a>
## getLightLSB()
#### Description
The method retrieves least significant byte of the raw sensor value from recent measurement. It is suitable for testing purposes or at very restrictive data transfers, e.g., over the air.

#### Syntax
    uint8_t getLightLSB();

#### Parameters
None

#### Returns
Low byte of the recently measured sensor value.

#### See also
[getLight()](#getLight)

[Back to interface](#interface)
