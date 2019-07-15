# **MAX30101 Photoplethymography (PPG) Sensor**
Driver Library for Arduino and Simplelink Microcontrollers

## Table of Contents
######[The MAX30101 PPG Sensor](#the-max30101-ppg-sensor)
######[Using the MAX30101ACCEVKIT with an External Microcontroller](#using-the-max30101accevkit-with-an-external-microcontroller)


## The MAX30101 PPG Sensor
The MAX30101 sensor is produced by Maxim Integrated and is designed in biomedical applications for the detection of heart rate and blood oxygen saturation (SpO2).

The sensor consists of three light emitting diodes (LEDs) emitting in the red (660nm), infrared (IR) (880nm) and green (537nm) parts of the electromagnetic spectrum. The sensor contains a single photodiode for detection of light emitted from the LEDs, or ambient light. This PPG sensor along with it's bi-wavelength companion sensor, the MAX30102, are the only PPG sensors with integrated analogue to digital converter (ADC) and I2C interface inside the sensor iteself. These features make the MAX30101 and MAX30102 sensors the smallest fully integrated PPG sensors on the market at about the size of a grain of rice.

## Using the MAX30101ACCEVKIT with an External Microcontroller
The development board for the MAX30101 is the MAX30101ACCEVKIT. This development kit contains a small board containing the MAX30101 sensor, included on this board is a 3-axis accelerometer, the LIS2DH. The kit includes a seperate board containing power regulators which step down USB voltage from 5v, to 4.4v, 3.3v and 1.8v required of the sensor board. The 4.4v is wired to the MAX30101 LEDs, the 1.8v is wired to the VDD of the MAX30101, while the 3.3v is wired to the VDD of the accelerometer. Additionally this board contains a microprocessor with custom firmware for intergration with the MAX30101ACCEVKIT software supplied by Maxim Integrated for testing from a Windows computer. Unlike the MAX30102, Maxim Integrated has not supplied demo code for the MAX30101 for Arduino or similar microprocessors, nor is the board included in the MAX30101ACCEVKIT immediately friendly for use with alternative microprocessors.

However, the included daugher board to the sensor board in the MAX30101ACCEVKIT can be made to work with an external microcontroller. In order to make this board work, you need to make a jumper to short the reset pin (pin 1) on J2 to ground (pin 3) on J2 (**Figure 1**). This will hold the onboard microcontroller in reset, allowing an external board to communicate via I2C with the sensors.

**INSERT FIGURE HERE WITH RESET PIN SHORTING**

Additionally, pins need to be added to the daugher board at SDA, SCL, INT1 (if desired), INT2 (if desired) GND and VBUS (5V in, if desired) (**Figure 2**). The SDA and SCL pins need to be connected to the SDA and SCL pins respectively on your microcontroller. Connect GND to the ground on your microcontroller (important, as both must have a common ground). If you want to power the daughter board from your microcontroller, connect the 5V or USB Power pin to the VBUS pin on the daughter board to supply 5V power. If you choose, you can omit the VBUS pin, however you must then power the daugher borad seperatly from your microcontroller, such as through a seperate USB power supply. This is stepped down on the daughter board to the voltages required of the sensor board as mentioned above. If desired the INT1 can be connected to a pin on your microcontroller for physical interrrupt for the MAX30101 sensor, however it is not necessary, nor is it used in this implementation of the driver. Additionally INT2 can be wired to your microcontroller for the acceleromoter physical interrupt. Please see the LIS2DH library for information on INT2.

**INSERT FIGURE HERE WITH SDA, SCL, VBUS, INT1 AND INT2 PIN LOCATIONS**

# Common Library Functions

The following are common to both the Arduino and Simplelink libraries

## The Initialiser
In the MAX30101 library a class named *Initialiser* is used to define the initialisation options for the MAX30101 sensor. The following sections describe how to create an * *Initialiser* * object, and the options available for initialisation of the MAX30101 sensor.

### Create an Initialiser Object
An *Initialiser* object is required in order to set the initialisation options of the MAX30101 sensor. This object is then passed into the Initialise function of the library configuring the MAX30101 sensor for use. To create an *Initialiser* function use the following syntax.
```
MAX30101::Initialiser ObjectName;
```
For example:
```
MAX30101::Initialiser initOptions;
```

Note: Further examples in this document will assume the *Initialiser* object is named *initOptions*.

### Sampling Average
The *Sampling Average* initialisation option sets the number of samples that are averaged to produce a single sample output from the MAX30101 sensor. Samples that are used to generate the averaged sample are not sent from the MAX30101 sensor, only the averaged value. If you have a sample rate of 100hz, and sampling average set to 8, then you will have an effective sampling rate of 12.5hz as you will only receive 12.5 samples per second from the MAX30101 sensor.

Valid values for *Sampling Average* are 1 (no averaging), 2, 4, 8, 16 and 32.

To set the *Sampling Average* use the following syntax.
```
initOptions.SampAvg(uint8_t);
```
For example:
```
initOptions.SampAvg(8);
```

### Buffer Rollover
The *Buffer Rollover* option defines if the buffer should rollover when full, and overwrite unread values, or if it should stop generating new data and wait until slots in the buffer have been freed.

Valid values for *Buffer Rollover* are true (rollover on) and false (rollover off);

To set the *Buffer Rollover* use the following syntax.
```
initOptions.FIFORollover(bool);
```
For example:
```
initOptions.FIFORollover(true);
```

### Buffer Almost Full Flag
The *Buffer Almost Full Flag* is an interrupt that is flagged when the data buffer is almost full. The number of free samples used to trigger the buffer almost full flag can be set. If the *Buffer Almost Full Flag* is set to 10 samples, then the *Buffer Almost Full Flag* interrupt will be triggered when there is 10 free slots left in the data buffer. Since the data buffer is 32 samples in size, the *Buffer Almost Full Flag* would be triggered when there is 22 samples in the buffer.

Valid values for *Buffer Almost Full Flag* are 0 to 15.

To set the *Buffer Almost Full Flag* use the following syntax.
```
initOptions.FIFOBuffFull(uint8_t);
```
For example:
```
initOptions.FIFOBuffFull(10);
```

### Mode Control
The *Mode Control* option is used to define the mode of operation for the MAX30101 sensor. There is three modes of operation defined below.

Valid values for *Mode Control* are HR, SPO2 and MULTI.

- **HR** - HR mode is used for recording heart rate only from the MAX30101 sensor. This mode uses only the red LED, all other LEDs are disbaled. Using this mode you do not need to set the *Multi LED Slots*.
- **SPO2** - SPO2 mode is used for recording heart rate and blood oxygen saturation (SPO2) from the MAX30101 sensor. This mode uses only the red and IR LEDs, the green LED is disabled.  Using this mode you do not need to set the *Multi LED Slots*.
- **MULTI** - MULTI mode allows custom configuration of the LEDs, enabling use of the green LED alone or along side of the red and IR LEDs. Using this mode you **must** configure the *Multi LED Slots* and *LED Pulse Amplitude* options.

To set the *Mode Control* option, use the following syntax.
```
initOptions.ModeCtrl(char*);
```
For example:
```
initOptions.ModeCtrl("MULTI");
```

### SPO2 ADC Range
The *SPO2 ADC Range* defined the range of the analogue to digital (ADC) in the MAX30101 sensor. The MAX30101 has a resolution of 18-bits.

Valid valuse for the *SPO2 ADC Range* are 2048, 4096, 8192 and 16384.

| LSB Size (pA) | Full Scale (nA) |
| :---: | :---: |
| 7.81 | 2048 |
| 15.63 | 4096 |
| 31.25 | 8192 |
| 62.5 | 16384 |

To set the *SPO2 ADC Range* use the following syntax.
```
initOptions.SPO2ADCRange(uint16_t);
```
For example:
```
initOptions.SPO2ADCRange(8192);
```

### SPO2 Sample Rate
The *SPO2 Sample Rate* set the sample rate in Hz. Note that if *Sampling Average* is configured higher than 1, then the effective sample rate at your microcontroller will be lower than this value. See the *[Sampling Average](#sampling-average)* section above for more information.

Also note that when setting the sample rate it is important to take into consideration the number of LEDs in use, the *[LED Pulse Width](#led-pulse-width)*. The *SPO2 Sample Rate* sets the upper bounds of the pulse width time, for example using a single LED at 100Hz, the maximum pulse width time could be 10ms. The higher the sample rate, the shorter the pulse width. At 3200Hz, the maximum pulse width with a single LED would be 312µs, while with 3 LED slots in use the maximum pulse width would be reduced to 104µs. **LINK TO SECTION ON THIS**

Valid values for *SPO2 Sample Rate* are 50, 100, 200, 400, 800, 1000, 1600 and 3200.

To set the *SPO2 Sample Rate* use the following syntax.
```
initOptions.SPO2SampRate(uint16_t);
```
For example:
```
initOptions.SPO2SampRate(100);
```

### LED Pulse Width
The *LED Pulse Width* sets the lenght of time in µs the LEDs can remain illuminated. Each if the LEDs (red, IR and green) have the same pulse width. The upper limit of the *LED Pulse Width* is linked to the SPO2 Sample Rate and the number of LEDs in use. Additionally, the pulse width indirectly sets the ADC Resolution of each sample as shown in the table below.

Valid valuse for *LED Pulse Width* are 69µs, 118µs, 215µs and 411µs.

| Pulse Width (Actual) | ADC Resolution |
| :---: | :---: |
| 69µs (68.95µs) | 15 bits |
| 118µs (117.78µs) | 16 bits |
| 215µs (215.44µs) | 17 bits |
| 411µs (410.75µs) | 18 bits |

To set the *LED Pulse Width* use the following syntax.
```
initOptions.LEDPulseWidth(uint16_t);
```
For example:
```
initOptions.LEDPulseWidth(118);
```
