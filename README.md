# MeshCore — RAK3401 1W Repeater with RAK12032 Accelerometer

A fork of [MeshCore](https://github.com/meshcore-dev/MeshCore) that adds 3-axis
accelerometer support to the **RAK3401 1W** repeater so you can tell how a node is
sitting — and detect if it has been knocked over — straight from the **Repeater Admin**
console in the MeshCore app (or over USB serial).

![sensor list showing accel_x/y/z and orientation in the Repeater Admin console](docs/images/rak12032-orientation-cli.jpg)

## Hardware

| Module | Part | Notes |
|--------|------|-------|
| Core   | **RAK3401** (RAK3400 stamp, Nordic nRF52840) | BLE-only WisBlock core, no built-in LoRa |
| LoRa   | **RAK13302** (SX1262 + SKY66122 PA, up to 1W) | On the WisBlock IO slot; needs a stable 5V supply |
| Sensor | **RAK12032** (Analog Devices **ADXL313**) | 3-axis accelerometer, I2C address `0x1D`, on the SENSOR slot |

Build target: **`RAK_3401_repeater`** (variant `variants/rak3401`).

```sh
pio run -e RAK_3401_repeater
# flash the resulting .pio/build/RAK_3401_repeater/firmware.zip via the MeshCore
# web flasher, or: adafruit-nrfutil dfu serial -pkg firmware.zip -p <port> -b 115200 -t 1200
```

## Reading orientation

Send these commands from the app's **Repeater Admin** console (log in with the admin
password) or over USB serial (`pio device monitor -p <port> -b 115200`):

| Command | Example reply |
|---------|---------------|
| `sensor list` | all four values at once |
| `sensor get accel_x` | `> -0.031` |
| `sensor get accel_y` | `> -0.293` |
| `sensor get accel_z` | `> -0.980` |
| `sensor get orientation` | `> 5 (upside down)` |

`accel_x/y/z` are in **g** (gravity ≈ ±1.0 on whichever axis points up/down).

### Orientation codes

Relative to the board lying flat on a desk:

| Code | Meaning |
|------|---------|
| 0 | straight up (flat) |
| 1 | left side |
| 2 | right side |
| 3 | forward |
| 4 | back |
| 5 | upside down |
| 255 | in-between (tilted between faces) |

## Telemetry note

MeshCore telemetry is transported as **Cayenne LPP**, a binary numeric-only format, and the
MeshCore app's telemetry view renders only a fixed subset of LPP types (temperature, humidity,
pressure, voltage, current, battery). The raw LPP **accelerometer** (type 113) and
**digital-input** types are not rendered, so accelerometer data does not appear in the normal
telemetry view. This feature therefore surfaces the readings through the **`sensor` CLI**, which
is the reliable readout. For completeness the firmware also emits the values on telemetry
channels 2–5 encoded as `temperature` (ch2=X, ch3=Y, ch4=Z, ch5=orientation code) in case a
future app build renders extra channels.

## Changes from upstream

- `variants/rak3401/platformio.ini` — enables `ENV_INCLUDE_RAK12032` and pulls in the
  `SparkFun ADXL313 Arduino Library`.
- `src/helpers/sensors/EnvironmentSensorManager.{h,cpp}` — detects the ADXL313 at `0x1D`
  (±2g range), computes the orientation, exposes `accel_x/y/z` + `orientation` via the `sensor`
  CLI, and includes the values in telemetry.

> The axis → direction mapping (left/right/forward/back) depends on how the module is
> physically mounted. The up/flat and upside-down axis is confirmed; flip the signs/cases in
> `accelOrientationCode()` if a horizontal direction comes out wrong for your mounting.

---

Based on MeshCore (repeater-v1.15.0). For the full MeshCore project, docs, and licensing,
see the [upstream repository](https://github.com/meshcore-dev/MeshCore).
