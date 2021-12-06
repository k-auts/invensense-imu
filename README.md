[![Pipeline](https://gitlab.com/bolderflight/software/mpu9250/badges/main/pipeline.svg)](https://gitlab.com/bolderflight/software/mpu9250/) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

![Bolder Flight Systems Logo](img/logo-words_75.png) &nbsp; &nbsp; ![Arduino Logo](img/arduino_logo_75.png)

# Mpu9250
This library communicates with [InvenSense MPU-9250 and MPU-9255](https://invensense.tdk.com/products/motion-tracking/9-axis/mpu-9250/) Inertial Measurement Units (IMUs). This library is compatible with Arduino and CMake build systems.
   * [License](LICENSE.md)
   * [Changelog](CHANGELOG.md)
   * [Contributing guide](CONTRIBUTING.md)

# Description
The InvenSense MPU-9250 is a System in Package (SiP) that combines two chips: the MPU-6500 three-axis gyroscope and three-axis accelerometer; and the AK8963 three-axis magnetometer. The MPU-9250 supports I2C, up to 400 kHz, and SPI communication, up to 1 MHz for register setup and 20 MHz for data reading. The following selectable full scale sensor ranges are available:

| Gyroscope Full Scale Range | Accelerometer Full Scale Range | Magnetometer Full Scale Range |
| --- | --- | ---  |
| +/- 250 deg/s  | +/- 2g  | +/- 4800 uT |
| +/- 500 deg/s  | +/- 4g  | |
| +/- 1000 deg/s | +/- 8g  | |
| +/- 2000 deg/s | +/- 16g | |

The MPU-9250 samples the gyros, accelerometers, and magnetometers with 16 bit analog to digital converters. It also features programmable digital filters, a precision clock, an embedded temperature sensor, and programmable interrupts.

# Installation

## Arduino
Use the Arduino Library Manager to install this library or clone to your Arduino/libraries folder. This library is added as:

```C++
#include "mpu9250.h"
```

Example Arduino executables, using I2C and SPI communication, are located at *examples/arduino/i2c_example/i2c_example.ino* and *examples/arduino/spi_example/spi_example.ino. Teensy 3.x, 4.x, and LC devices are used for testing under Arduino and this library should be compatible with other Arduino devices.

## CMake
CMake is used to build this library, which is exported as a library target called *mpu9250*. The header is added as:

```C++
#include "mpu9250.h"
```

The library can be also be compiled stand-alone using the CMake idiom of creating a *build* directory and then, from within that directory issuing:

```
cmake .. -DMCU=MK66FX1M0
make
```

This will build the library and example executables called *i2c_example* and *spi_example*. The example executable source files are located at *examples/cmake/i2c.cc* and *examples/cmake/spi.cc*. Notice that the *cmake* command includes a define specifying the microcontroller the code is being compiled for. This is required to correctly configure the code, CPU frequency, and compile/linker options. The available MCUs are:
   * MK20DX128
   * MK20DX256
   * MK64FX512
   * MK66FX1M0
   * MKL26Z64
   * IMXRT1062_T40
   * IMXRT1062_T41

These are known to work with the same packages used in Teensy products. Also switching packages is known to work well, as long as it's only a package change.

The *i2c_example* and *spi_example* targets create executables for communicating with the sensor using I2C or SPI communication, respectively. Each target also has a *_hex*, for creating the hex file to upload to the microcontroller, and an *_upload* for using the [Teensy CLI Uploader](https://www.pjrc.com/teensy/loader_cli.html) to flash the Teensy. Please note that the CMake build tooling is expected to be run under Linux or WSL, instructions for setting up your build environment can be found in our [build-tools repo](https://github.com/bolderflight/build-tools).

# Namespace
This library is within the namespace *bfs*.

# Mpu9250

## Methods

**Mpu9250(i2c_t3 &ast;bus, uint8_t addr)** Creates a Mpu9250 object. This constructor is used for the I2C communication interface. A pointer to the I2C bus object is passed along with the I2C address of the sensor. The address will be 0x68 if the AD0 pin is grounded and 0x69 if the AD0 pin is pulled high.

```C++
Mpu9250 mpu9250(&Wire, 0x68);
```

**Mpu9250(SPIClass &ast;bus, uint8_t cs)** Creates a Mpu9250 object. This constructor is used for the SPI communication interface. A pointer to the SPI bus object is passed along with the chip select pin of the sensor. Any pin capable of digital I/O can be used as a chip select pin.

```C++
Mpu9250 mpu9250(&SPI, 2);
```

**bool Begin()** Initializes communication with the sensor and configures the default sensor ranges, sampling rates, and low pass filter settings. The default accelerometer range is +/- 16g and the default gyro range is +/- 2,000 deg/s. The default sampling rate is 1000 Hz and the low-pass filter is set to a cutoff frequency of 184 Hz. True is returned if communication is able to be established with the sensor and configuration completes successfully, otherwise, false is returned. The communication bus is not initialized within this library and must be initialized seperately; this enhances compatibility with other sensors that may on the same bus.

```C++
Wire.begin();
Wire.setClock(400000);
bool status = mpu9250.Begin();
if (!status) {
  // ERROR
}
```

**bool EnableDrdyInt()** Enables the data ready interrupt. A 50 us interrupt will be triggered on the MPU-9250 INT pin when IMU data is ready. This interrupt is active high. This method returns true if the interrupt is successfully enabled, otherwise, false is returned.

```C++
bool status = mpu9250.EnableDrdyInt();
if (!status) {
  // ERROR
}
```

**bool DisableDrdyInt()** Disables the data ready interrupt. This method returns true if the interrupt is successfully disabled, otherwise, false is returned.

```C++
bool status = mpu9250.DisableDrdyInt();
if (!status) {
  // ERROR
}
```

**bool ConfigAccelRange(const AccelRange range)** Sets the accelerometer full scale range. Options are:

| Range | Enum Value |
| --- | --- |
| +/- 2g | ACCEL_RANGE_2G |
| +/- 4g | ACCEL_RANGE_4G |
| +/- 8g | ACCEL_RANGE_8G |
| +/- 16g | ACCEL_RANGE_16G |

True is returned on succesfully setting the accelerometer range, otherwise, false is returned. The default range is +/-16g.

```C++
bool status = mpu9250.ConfigAccelRange(Mpu9250::ACCEL_RANGE_4G);
if (!status) {
  // ERROR
}
```

**AccelRange accel_range()** Returns the current accelerometer range.

```C++
AccelRange range = mpu9250.accel_range();
```

**bool ConfigGyroRange(const GyroRange range)** Sets the gyro full scale range. Options are:

| Range | Enum Value |
| --- | --- |
| +/- 250 deg/s | GYRO_RANGE_250DPS |
| +/- 500 deg/s | GYRO_RANGE_500DPS |
| +/- 1000 deg/s | GYRO_RANGE_1000DPS |
| +/- 2000 deg/s | GYRO_RANGE_2000DPS |

True is returned on succesfully setting the gyro range, otherwise, false is returned. The default range is +/-2000 deg/s.

```C++
bool status = mpu9250.ConfigGyroRange(Mpu9250::GYRO_RANGE_1000DPS);
if (!status) {
  // ERROR
}
```

**GyroRange gyro_range()** Returns the current gyro range.

```C++
GyroRange range = mpu9250.gyro_range();
```

**bool ConfigSrd(const uint8_t srd)** Sets the sensor sample rate divider. The MPU-9250 samples the accelerometer and gyro at a rate, in Hz, defined by:

```math
rate = 1000 / (srd + 1)
```

A *srd* setting of 0 means the MPU-9250 samples the accelerometer and gyro at 1000 Hz. A *srd* setting of 4 would set the sampling at 200 Hz. The IMU data ready interrupt is tied to the rate defined by the sample rate divider. The magnetometer is sampled at 100 Hz for sample rate divider values corresponding to 100 Hz or greater. Otherwise, the magnetometer is sampled at 8 Hz.

True is returned on succesfully setting the sample rate divider, otherwise, false is returned. The default sample rate divider value is 0, resulting in a 1000 Hz sample rate.

```C++
/* Set sample rate divider for 50 Hz */
bool status = mpu9250.sample_rate_divider(19);
if (!status) {
  // ERROR
}
```

**uint8_t srd()** Returns the current sample rate divider value.

```C++
uint8_t srd = mpu9250.srd();
```

**bool ConfigDlpf(const DlpfBandwidth dlpf)** Sets the cutoff frequency of the digital low pass filter for the accelerometer, gyro, and temperature sensor. Available bandwidths are:

| DLPF Bandwidth | Enum Value |
| --- | --- |
| 184 Hz | DLPF_BANDWIDTH_184HZ |
| 92 Hz | DLPF_BANDWIDTH_92HZ |
| 41 Hz | DLPF_BANDWIDTH_41HZ |
| 20 Hz | DLPF_BANDWIDTH_20HZ |
| 10 Hz | DLPF_BANDWIDTH_10HZ |
| 5 Hz | DLPF_BANDWIDTH_5HZ |

True is returned on succesfully setting the digital low pass filters, otherwise, false is returned. The default bandwidth is 184 Hz.

```C++
bool status = mpu9250.ConfigDlpf(Mpu9250::DLPF_BANDWIDTH_20HZ);
if (!status) {
  // ERROR
}
```

**DlpfBandwidth dlpf()** Returns the current digital low pass filter bandwidth setting.

```C++
DlpfBandwidth dlpf = mpu9250.dlpf();
```

**bool Read()** Reads data from the MPU-9250 and stores the data in the Mpu9250 object. Returns true if data is successfully read, otherwise, returns false.

```C++
/* Read the IMU data */
if (mpu9250.Read()) {
}
```

**bool new_imu_data()** Returns true if new data was returned from the accelerometer and gyro.

```C++
if (mpu9250.Read()) {
  bool new_data = mpu9250.new_imu_data();
}
```

**bool new_mag_data()** Returns true if new data was returned from the magnetometer. For MPU-9250 sample rates of 100 Hz and higher, the magnetometer is sampled at 100 Hz. For MPU-9250 sample rates less than 100 Hz, the magnetometer is sampled at 8 Hz, so it is not uncommon to receive new IMU data, but not new magnetometer data.

```C++
if (mpu9250.Read()) {
  bool new_mag = mpu9250.new_mag_data();
}
```

**float accel_x_mps2()** Returns the x accelerometer data from the Mpu9250 object in units of m/s/s. Similar methods exist for the y and z axis data.

```C++
/* Read the IMU data */
if (mpu9250.Read()) {
  float ax = mpu9250.accel_x_mps2();
  float ay = mpu9250.accel_y_mps2();
  float az = mpu9250.accel_z_mps2();
}
```

**float gyro_x_radps()** Returns the x gyro data from the Mpu9250 object in units of rad/s. Similar methods exist for the y and z axis data.

```C++
/* Read the IMU data */
if (mpu9250.Read()) {
  float gx = mpu9250.gyro_x_radps();
  float gy = mpu9250.gyro_y_radps();
  float gz = mpu9250.gyro_z_radps();
}
```

**float mag_x_ut()** Returns the x magnetometer data from the Mpu9250 object in units of uT. Similar methods exist for the y and z axis data.

```C++
/* Read the IMU data */
if (mpu9250.Read()) {
  float hx = mpu9250.mag_x_ut();
  float hy = mpu9250.mag_y_ut();
  float hz = mpu9250.mag_z_ut();
}
```

**float die_temp_c()** Returns the die temperature of the sensor in units of C.

```C++
/* Read the IMU data */
if (mpu9250.Read()) {
  float temp = mpu9250.die_temp_c();
}
```

## Sensor Orientation
This library transforms all data to a common axis system before it is returned. This axis system is shown below. It is a right handed coordinate system with the z-axis positive down, common in aircraft dynamics.

<img src="https://github.com/bolderflight/mpu9250/blob/main/docs/MPU-9250-AXIS.png" alt="Common Axis System" width="250">

**Caution!** This axis system is shown relative to the MPU-9250 sensor. The sensor may be rotated relative to the breakout board. 

# Example List

* **I2C**: demonstrates declaring an *Mpu9250* object, initializing the sensor, and collecting data. I2C is used to communicate with the MPU-9250 sensor.
* **SPI**: demonstrates declaring an *Mpu9250* object, initializing the sensor, and collecting data. SPI is used to communicate with the MPU-9250 sensor.

# Wiring and Pullups 

Please refer to the [MPU-9250 datasheet](https://github.com/bolderflight/mpu9250/blob/main/docs/MPU-9250-Datasheet.pdf) and your microcontroller's pinout diagram. This library was developed using the Embedded Masters breakout board v1.1 for the MPU-9250. The data sheet for this breakout board is located [here](https://github.com/bolderflight/mpu9250/blob/main/docs/Embedded-Masters-MPU-9250-Breakout.pdf). This library should work well for other breakout boards or embedded sensors, please refer to your vendor's pinout diagram.

## I2C

The MPU-9250 pins should be connected as:
   * VDD: this should be a 2.4V to 3.6V power source.
   * GND: ground.
   * VDDI: digital I/O supply voltage. This should be between 1.71V and VDD.
   * FSYNC: not used, should be grounded.
   * INT: (optional) used for the interrupt output setup in *enableDataReadyInterrupt*. Connect to interruptable pin on microcontroller.
   * SDA / SDI: connect to SDA.
   * SCL / SCLK: connect to SCL.
   * AD0 / SDO: ground to select I2C address 0x68. Pull high to VDD to select I2C address 0x69.
   * nCS: no connect.
   * AUXDA: not used.
   * AUXCL: not used.

2.2 kOhm resistors should be used as pullups on SDA and SCL, these resistors should pullup with a 3.3V source.

## SPI

The MPU-9250 pins should be connected as:
   * VDD: this should be a 2.4V to 3.6V power source.
   * GND: ground.
   * VDDI: digital I/O supply voltage. This should be between 1.71V and VDD.
   * FSYNC: not used, should be grounded.
   * INT: (optional) used for the interrupt output setup in *enableDataReadyInterrupt*. Connect to interruptable pin on microcontroller.
   * SDA / SDI: connect to MOSI.
   * SCL / SCLK: connect to SCK.
   * AD0 / SDO: connect to MISO.
   * nCS: connect to chip select pin. Pin 10 was used in the code snippets in this document and the included examples, but any digital I/O pin can be used. 
   * AUXDA: not used.
   * AUXCL: not used.

Some breakout boards, including the Embedded Masters breakout board, require slight modification to enable SPI. Please refer to your vendor's documentation.