# DS18B20

Arduino library for the Maxim Integrated DS18B20 1-Wire temperature sensor. Supports auto-discovering sensors on a bus, alarm filtering, and manually addressing individual sensors.

## Usage

### Auto-scan all sensors

```cpp
#include <DS18B20.h>

DS18B20 ds{2};  // 1-Wire data pin

void loop() {
    while (ds.selectNext()) {
        std::optional<float> temp{ds.getTempC()};
        if (temp.has_value()) {
            Serial.println(*temp);
        }
    }
}
```

### Address a specific sensor

```cpp
uint8_t address[8];
ds.getAddress(address);       // Save after selectNext()

ds.select(address);           // Select by address on future reads
std::optional<float> temp{ds.getTempC()};
```

### Alarm scan

```cpp
while (ds.selectNextAlarm()) {
    std::optional<float> temp{ds.getTempC()};
    if (temp.has_value()) {
        Serial.println(*temp);
    }
}
```

## API

### Selection

| Method | Description |
|---|---|
| `selectNext()` | Advances to the next sensor on the bus. Returns 0 when none remain. |
| `selectNextAlarm()` | Advances to the next sensor with an active alarm condition. |
| `select(const uint8_t address[])` | Selects a specific sensor by 8-byte ROM address. |
| `resetSearch()` | Resets the search so `selectNext()` starts from the first device again. |
| `getNumberOfDevices()` | Returns the total number of DS18B20s found during the last scan. |

### Temperature

`getTempC()` and `getTempF()` return `std::optional<float>`. `std::nullopt` is returned if the 1-Wire command or scratchpad read fails — always check before using the value.

| Method | Description |
|---|---|
| `getTempC()` | Temperature in °C, or `std::nullopt` on read failure. |
| `getTempF()` | Temperature in °F, or `std::nullopt` on read failure. |
| `doConversion()` | Issues a broadcast conversion command to all sensors simultaneously (use `getTempC()` for single-sensor reads). |

### Resolution

| Method | Description |
|---|---|
| `getResolution()` | Returns current resolution: 9, 10, 11, or 12 bits. |
| `setResolution(uint8_t)` | Sets resolution (9–12 bits). Higher resolution increases conversion time. |

Conversion times: 9-bit = 94 ms, 10-bit = 188 ms, 11-bit = 375 ms, 12-bit = 750 ms.

### Alarms

| Method | Description |
|---|---|
| `hasAlarm()` | Returns `std::optional<bool>` — `true` if current temp is outside alarm bounds, `std::nullopt` if the read failed. |
| `setAlarms(int8_t low, int8_t high)` | Sets both alarm thresholds. |
| `getAlarmLow()` / `setAlarmLow(int8_t)` | Low alarm threshold (°C). |
| `getAlarmHigh()` / `setAlarmHigh(int8_t)` | High alarm threshold (°C). |

`setRegisters` / `getLowRegister` / `getHighRegister` are aliases for the alarm methods.

### Other

| Method | Description |
|---|---|
| `getPowerMode()` | Returns power mode of the selected sensor (0 = parasitic, 1 = external). |
| `getFamilyCode()` | Returns the 1-Wire family code byte. |
| `getAddress(uint8_t[])` | Copies the 8-byte address of the selected sensor into the provided buffer. |

## Dependencies

- [OneWire](https://github.com/PaulStoffregen/OneWire)

## Wiring

Pull-up resistor: **4.7 kΩ** on the data line in all configurations.

### External power

![Single externally powered DS18B20](/extras/single_external.png)
![Multiple externally powered DS18B20s](/extras/multiple_external.png)

### Parasitic power

![Single parasite powered DS18B20](/extras/single_parasite.png)
![Multiple parasite powered DS18B20s](/extras/multiple_parasite.png)

### Mixed

![Mixed mode DS18B20s](/extras/mixed_mode.png)
