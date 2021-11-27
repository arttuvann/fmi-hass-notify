# fmi-hass-notify
This is documentation for setting up FMI (Ilmatieteenlaitos) integration and mobile notifications to HASS.

    OS Version:               Home Assistant OS 6.6
    Home Assistant Core:      2021.11.5

Use your preferred method such as Terminal & SSH login to upload files from repository to HASS.

```bash
mkdir -p /config/custom_components/fmi
cd <downloaded_folder>
scp * root@<HASS IP>:/config/custom_components/fmi
```

Add the following line to configuration.yaml:

```yaml
sensor: !include sensor.yaml
```

Create sensor.yaml and add the following:

```yaml
- platform: template
  sensors:
    <custom name for sensor, e.g. my_morning_weather>:
      friendly_name: "Weather Conditions in the Morning"
      entity_id: weather.<your location>
      unit_of_measurement: '°C'
      value_template: "Lämpötilaennuste {{ as_timestamp(state_attr('weather.<your location>', 'forecast')[9]['datetime']) | timestamp_custom('%d.%m.%y') }}:
  

      {{ as_timestamp(state_attr('weather.<your location>', 'forecast')[9]['datetime']) | timestamp_custom('klo %H:%M') }}:
      {{ state_attr('weather.<your location>', 'forecast')[9]['temperature']}} °C
  

      {{ as_timestamp(state_attr('weather.<your location>', 'forecast')[10]['datetime']) | timestamp_custom('klo %H:%M') }}:
      {{ state_attr('weather.<your location>', 'forecast')[10]['temperature']}} °C
  

      {{ as_timestamp(state_attr('weather.<your location>', 'forecast')[11]['datetime']) | timestamp_custom('klo %H:%M') }}:
      {{ state_attr('weather.<your location>', 'forecast')[11]['temperature']}} °C"
```

This creates a sensor that produces the forecast text from the FMI weather entity. I created this sensor to be part of the automation thatis run at 9 PM, and so it creates forecast text for temperatures 9, 10 and 11 hours from now (6, 7 and 8 AM).

Your needs may vary, and checking Developer Tools for the FMI Weather entity is simple. Example data:

```yaml
forecast:
  - datetime: '2021-11-27T11:00:00+02:00'
    condition: sunny
    temperature: -11.1
    precipitation: 0
    wind_speed: 1.32
    wind_bearing: 51
    pressure: 1007.6
    humidity: 88.81
  - datetime: '2021-11-27T12:00:00+02:00'
    condition: sunny
    temperature: -9.9
    precipitation: 0
    wind_speed: 1.43
    wind_bearing: 59
    pressure: 1007.26
    humidity: 86.13

    ...etc
```

and you could access the second item's attributes with the template

```yaml
{{ state_attr('weather.<your location>', 'forecast')[1]['condition'] }}
```

Adding also the timestamp is beneficial for making sure you're reading the right hour's data.

Mobile notification can be created with the desired execution time and action: 

    Action type: Call service
    Service: Notifications: Send a notification via... (your mobile device id here)

Click on the three dots to edit YAML directly (templates are not supported in visual editor):

```yaml
service: notify.mobile_<check your own HASS mobile device id>
data:
  message: '{{ states(''sensor.my_morning_weather'') }}'
  title: Morning forecast
```

There you have it. Enjoy your automated weather notifications based on FMI data :)