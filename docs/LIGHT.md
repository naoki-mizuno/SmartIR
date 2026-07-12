# SmartIR Light

Find your device's brand code [here](LIGHT_CODES.md) and add the number in the `device_code` field. If your device is not supported, you will need to learn your own IR codes and place them in the Json file in `smartir/custom_codes/light` subfolder. Please refer to [this guide](CODES_SYNTAX.md) to find a way how to do it. Once you have working device file please do not forgot to submit Pull Request so it could be inherited to this project for other users.

## Configuration variables

| Name                         |  Type   | Default  | Description                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------- | :-----: | :------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                       | string  | optional | The name of the device                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `unique_id`                  | string  | optional | An ID that uniquely identified this device. If two devices have the same unique ID, Home Assistant will raise an exception.                                                                                                                                                                                                                                                                                                               |
| `device_code`                | number  | required | (Accepts only positive numbers)                                                                                                                                                                                                                                                                                                                                                                                                           |
| `controller_data`            | string  | required | The data required for the controller to function. Look into configuration examples below for valid configuration entries for different controller types.                                                                                                                                                                                                                                                                                  |
| `delay`                      | number  | optional | Adjusts the delay in seconds between multiple commands. The default is 0.5                                                                                                                                                                                                                                                                                                                                                                |
| `power_sensor`               | string  | optional | _entity_id_ for a sensor or that monitors whether your device is actually On or Off. This may be a power monitor sensor, or a helper that monitors power usage with a threshold. (Accepts only on/off states)                                                                                                                                                                                                                             |
| `power_sensor_delay`         |   int   | optional | Maximum delay in second in which power sensor is able to report back to HA changed state of the device, default is 10 seconds. If sensor reaction time is longer extend this time, otherwise you might get unwanted changes in the device state.                                                                                                                                                                                          |
| `power_sensor_restore_state` | boolean | optional | If `true` than in case power sensor will report to HA that device is `on` without HA actually switching it `on `(device was switched on by remote, of device cycled, etc.), than HA will report last assumed state and attributes at the time when the device was `on` managed by HA. If set to `false` when device will be reported as `on` by the power sensors all device attributes will be reported as `UNKNOWN`. Default is `true`. |
| `always_send_command`        | boolean | optional | If `true`, always send the `on`/`off` command, even if SmartIR's cached state already matches the requested state (normally skipped to avoid unnecessary/duplicate command sends). Use this if the cached state can drift from the real device state and commands stop getting sent. Default is `false`. |

### Shared controller options

Every `controller_data` type accepts an optional `command_delay` (number of seconds, e.g. `0.75`). Commands addressed to the same physical controller (for example one IR blaster shared by several SmartIR entities) are always sent one at a time; `command_delay` sets the pause after each command before the next one is sent. Use it for controllers such as the UFO-R11 which drop commands received while still transmitting the previous one.

## Example (using broadlink controller)

Add a Broadlink RM device named "Bedroom" via config flow (read the [docs](https://www.homeassistant.io/integrations/broadlink/)).

```yaml
light:
  - platform: smartir
    name: Bedroom Ceiling Light
    unique_id: bedroom_ceiling_light
    device_code: 1000
    controller_data:
      controller_type: Broadlink
      remote_entity: remote.bedroom_remote
      delay_secs: 0.5
      num_repeats: 1
    power_sensor: binary_sensor.bedroom_light_power
```

## Example (using xiaomi controller)

```yaml
remote:
  - platform: xiaomi_miio
    host: 192.168.10.10
    token: YOUR_TOKEN

light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 2000
    controller_data:
      controller_type: Xiaomi
      remote_entity: remote.xiaomi_miio_192_168_10_10
    power_sensor: binary_sensor.bedroom_light_power
```

### Example (using MQTT controller)

```yaml
light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 3000
    controller_data:
      controller_type: MQTT
      mqtt_topic: home-assistant/bedroom_light/command
    power_sensor: binary_sensor.bedroom_light_power
```

### Example (using mqtt Z06/UFO-R11 controller)

```yaml
light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 3000
    controller_data:
      controller_type: UFOR11
      mqtt_topic: home-assistant/bedroom_light/command
    power_sensor: binary_sensor.bedroom_light_power
```

### Example (using LOOKin controller)

```yaml
light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 4000
    controller_data:
      controller_type: LOOKin
      remote_host: 192.168.10.10
    power_sensor: binary_sensor.bedroom_light_power
```

### Example (using ESPHome)

ESPHome configuration example:

```yaml
esphome:
  name: my_espir
  platform: ESP8266
  board: esp01_1m

api:
  services:
    - service: send_raw_command
      variables:
        command: int[]
      then:
        - remote_transmitter.transmit_raw:
            code: !lambda "return command;"

remote_transmitter:
  pin: GPIO14
  carrier_duty_percent: 50%
```

HA configuration.yaml:

```yaml
light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 4000
    controller_data:
      controller_type: ESPHome
      esphome_service: my_espir_send_raw_command
    power_sensor: binary_sensor.bedroom_light_power
```

### Example (using ZHA controller and a TuYa ZS06)

```yaml
light:
  - platform: smartir
    name: Bedroom light
    unique_id: bedroom_light
    device_code: 5000
    controller_data:
      controller_type: ZHA
      zha_ieee: "XX:XX:XX:XX:XX:XX:XX:XX"
      zha_endpoint_id: 1
      zha_cluster_id: 57348
      zha_cluster_type: "in"
      zha_command: 2
      zha_command_type: "server"
    power_sensor: binary_sensor.bedroom_light_power
```

## Light device files

As well as the command mappings, the light device config supports two lists:
`brightness` and `color_temperature`. These should be sorted lists
from lower to higher values of brightness on a scale of 1 to 255, and
color temperature in Kelvin (normally from 2700 to 6500). Supported
commands are `on` `off`, `brighten`, `dim`, `warmer`, `colder` and `night`.
If `night` is configured, it is implemented as a special brightness step that
can be selected by setting a brightness of 1 (such lights usually have a
separate small and dim nightlight bulb inside the fixture).

## Available codes for Light devices

[**Light codes**](/docs/LIGHT_CODES.md)
