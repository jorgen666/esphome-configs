Howto get the Gill Windsonic Anemonemeter to work on ESPHome.

Hardware needed are:

1. Gill Windsonic option 1 sensor
2. RS-232 to TTL adapter


## Config
windsonic.YAML
```YAML
esphome:
  name: vind2
  platform: ESP32
  board: featheresp32
  includes:
    - windsonic.h
wifi:
  ssid: "ssid"
  password: "xxxxxxx"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Vind2 Fallback Hotspot"
    password: "xxxxxxxxx"

captive_portal:

# Enable logging
logger:
  baud_rate: 9600
  
# Enable Home Assistant API
api:
  password: "xxxxxxxx"
  reboot_timeout: 60s
  
ota:
  password: "xxxxxxxx"

uart:
  id: wind
  tx_pin: GPIO17
  rx_pin: GPIO16
  baud_rate: 19200
  
sensor:
#  - platform: uptime
#    name: Uptime sensor
    
  - platform: custom 
    lambda: |-
      auto my_windsonic = new Windsonic(id(wind));
      App.register_component(my_windsonic);
      return {my_windsonic->winddirection_sensor, my_windsonic->windspeed_sensor};
      
    sensors:
    - name: "Wind direction"
      unit_of_measurement: "gr"
      accuracy_decimals: 3
      icon: mdi:compass-rose
    - name: "Wind speed"
      unit_of_measurement: "ms"
      accuracy_decimals: 3
      icon: mdi:weather-windy 
 ```
 
 windsonic.h
 ```h
 #include "esphome.h"
#include "sensor.h"

class Windsonic : public Component, public UARTDevice {
 public:
  Windsonic(UARTComponent *parent) : UARTDevice(parent) {}
  Sensor *winddirection_sensor = new Sensor();
  Sensor *windspeed_sensor = new Sensor();
  
  
  
  void setup() override {
    // nothing to do here
  }
  void loop() override {
  // Use Arduino API to read data, for example
  //String line = "";  
  
  String line = readStringUntil('\n');
  String r = line.substring(7, 10);
	String s = line.substring(13, 19);
  float winddirection = r.toFloat();
	float windspeed = s.toFloat();
  winddirection_sensor->publish_state(winddirection);
  windspeed_sensor->publish_state(windspeed);
  winddirection = 0;
  windspeed = 0;
  r = "";
	s = "";
	
	
  }
  
};
```
