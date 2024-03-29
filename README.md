# Home Assistant - Localvolts
## Localvolts API integration with Home Assistant using Node-RED Companion

<img src="https://raw.githubusercontent.com/melvanderwal/HA-Localvolts/main/Localvolts_HA_Device.png" width="200">

<img src="https://raw.githubusercontent.com/melvanderwal/HA-Localvolts/main/Localvolts_HA_Flow.png" width="500">

### Steps
If you don't have Node-RED running in HA yet....

-   Install the Node-RED Add-on found in HACS.
-   Install the Node-RED Companion integration found in HACS.
-   Make a little flow and confirm that it's running ok.
-   There's stuff on the interwebz to help you get started, like [Beginners Guide to Node-RED in Home Assistant](https://www.youtube.com/watch?v=KXcwUdDqXXo&t=1403s).

Import _localvolts.json_ to a new flow in Node-RED.

-   This will create a new Home Assistant server configuration node, configured for use with the HA Node-RED add-on.  Using a standalone Node-RED instance rather than the add-on will require configuring the server node differently.
-   Hamburger menu > _Import_ > paste the contents of _localvolts.json_.

Open the _Localvolts_ flow and update the _Request Properties_ node with your:
-   NMI
-   Partner ID
-   API Key

Don't change _apiKeyFormatted_. Press _Done_ and deploy the flow.  A new device is added called _Localvolts Prices_.

Adjust the display precision of the device's sensors to your preference.

#### Notes
The forecast is stored in the _forecast_ attribute of the _Forecast_ sensor because the state of a sensor can only hold 255 characters.

Prices only refresh when the quality value is not _Forecast_ (i.e. it's _Actual_ or _Expected_).  Once they have refreshed, the Localvolts API will not be called until the current period ends, and then will be checked every 10 seconds until the quality value is no longer _Forecast_. 

Where a sensor's unit is $, a device class of monetary and unit of USD was used to prevent the preceding _A_ that appears when AUD is used.

#### Manual Price Refresh
You can manually refresh the prices from Home Assistant by hooking a button or tile up to a REST Command to call the _GET Refresh Data_ node. Update the url to one that works with your Node-RED setup.

REST Command example:
```
rest_command:
  electricity_refresh_prices:
    url: "http://homeassistant.local:1880/endpoint/localvolts/refresh"
    method: GET
```

Tile card example:

![image](https://github.com/melvanderwal/HA-Localvolts/assets/25993713/0440b2e0-bd03-4f22-81c0-055cfb96dee3)

```
type: tile
entity: sensor.last_checked
vertical: true
name: ' '
tap_action:
  action: call-service
  service: rest_command.electricity_refresh_prices
  target: {}
icon_tap_action:
  action: call-service
  service: rest_command.electricity_refresh_prices
  target: {}
icon: mdi:sync
color: light-green
state_content:
  - state
  - status
```

#### ApexCharts Card by @RomRider
You can use the ApexCharts Card frontend integration (found in HACS) to display forecast data.

Example:
![image](https://github.com/melvanderwal/HA-Localvolts/assets/25993713/59431e64-4beb-4a22-8242-63f200f2290f)


```
type: custom:apexcharts-card
update_delay: 3s
update_interval: 15s
graph_span: 24h
span:
  start: minute
show:
  last_updated: false
header:
  show: true
  title: Forecast
  show_states: false
apex_config:
  chart:
    height: 335px
  tooltip:
    enabled: true
    x:
      show: false
yaxis:
  - id: dollars
    decimals: 2
    min: ~-0
    max: ~0.5
    apex_config:
      forceNiceScale: true
      title:
        text: $
        rotate: 0
all_series_config:
  type: area
  curve: stepline
  stroke_width: 0
  extend_to: false
  yaxis_id: dollars
  float_precision: 2
  unit: $
  show:
    legend_value: false
    extremas: false
    in_header: false
series:
  - entity: sensor.forecast
    name: Buy Price
    color: firebrick
    data_generator: |
      return entity.attributes.forecast.map((forecast) => {
        return [new Date(forecast.intervalStart).getTime(), forecast.costsFlexUp];
      })
    show:
      extremas: min+time
  - entity: sensor.forecast
    name: Sell Price
    color: forestgreen
    data_generator: |
      return entity.attributes.forecast.map((forecast) => {
        return [new Date(forecast.intervalStart).getTime(), forecast.earningsFlexUp];
      })
    show:
      extremas: max+time
view_layout:
  position: main
```
