# Home Assistant - Localvolts
### Localvolts API integration with Home Assistant using Node-RED Companion.

#### Steps
If you don't have Node-RED running in HA yet....

-   Install the Node-RED Add-on.
-   Install the Node-RED Companion integration found in HACS.
-   Make a little flow and confirm that it's running ok.
-   There's stuff on the interwebz, like [Beginners Guide to Node-RED in Home Assistant](https://www.youtube.com/watch?v=KXcwUdDqXXo&t=1403s)

Import localvolts.json to a new flow in Node-RED.

-   This will create a new Home Assistant server configuration node, configured for use with the HA Node-RED add-on.  Using a standalone Node-RED instance will require a different server configuration node.
-   Hamburger menu > Import > paste the contents of localvolts.json.

Open the Localvolts flow and update the Request Properties node with your:

-   NMI
-   Partner ID
-   API Key
-   Don't change apiKeyFormatted
-   Press Done.

Deploy the flow.  A new device is added called Localvolts Prices.

Adjust the display precision of the device's sensors to your preference.


The forecast is stored in the forecast attribute of the Forecast sensor because the state of a sensor can only hold 255 characters.

Prices only refresh when the quality value is not Forecast (i.e. it's Actual or Expected).  Once they have refreshed, the Localvolts API will not be called until the current period ends, and then will be checked every 10 seconds until the quality value is no longer Forecast. 

You can manually refresh the prices from Home Assistant hooking a button or tile up to a REST Command to call the GET Refresh Data node.
rest_command:
  electricity_refresh_prices:
    url: "http://homeassistant.local:1880/endpoint/localvolts/refresh"
    method: GET

Tile example:
![image](https://github.com/melvanderwal/HA-Localvolts/assets/25993713/0440b2e0-bd03-4f22-81c0-055cfb96dee3)


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
