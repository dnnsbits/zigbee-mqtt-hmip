# hmip-mqtt-zigbee

## Voraussetzung

* HMIP CCU3
* ccu-jack https://github.com/mdzio/ccu-jack
* Zigbee Bridge (Sonoff ZB Bridge Pro) mit Tasmota https://tasmota.github.io/docs/Zigbee/
* MQTT Explorer https://github.com/thomasnordquist/MQTT-Explorer

## Einstellungen 

### ZB Bridge

* Einstellungen/MQTT Einstellungen
  * MQTT Server IP und Port: __192.168.1.200:1883__
  * Topic: __zbbridge__
  * Full Topic: __%topic%/%prefix%/__
* freundliches JSON (Name statt Geräte-ID)
  * Werkzeuge/Konsole: __SetOption83 1__
 
    `{"ZbReceived":{"0x8FD8":{"Device":"0x5480","Temperature":20.3,"Endpoint":1,"LinkQuality":76}}}`
    
    wird zu
    
    `{"ZbReceived":{"THERMO1":{"Device":"0x5480","Name":"THERMO1","Temperature":20.9,"Endpoint":1,"LinkQuality":76}}}`
 
* check im MQTT Explorer
  * Telemetriedaten Topic __zbbridge/tele__
    * __STATE__ über die Bridge
    * __SENSOR__ über verbundene Sensoren
   
## Geräte

### Sonoff SNZB-02D Temperatur- und Luftfeuchtigkeitssensor mit Display

#### ZB Bridge

* Gerät benennen -> Werkzeuge/Konsole: __ZbName 0x5480,THERMO1__

#### MQTT Topic 

Topic _zbbridge/tele/SENSOR_ Value ist _OHNE_ Leerzeichen und Umbrüche!

    {
      "ZbReceived": {
        "THERMO1": {
          "Device": "0x5480",
          "Name": "THERMO1",
          "Temperature": 20.9,
      	  "Humidity":40.6,
          "Endpoint": 1,
          "LinkQuality": 76
        }
      }
    }

#### ccu-jack

* Virtuelle Geräte/Erstellen
  * Gerätesymbol __HMIP-STHO__
  * Kanal: __MQTT Temperatursensor__
 
#### CCU3

* Geräte Einstellungen (für Kanal 1)
  * CLIMATE_TRANSCEIVER|TEMPERATURE_TOPIC __zbbridge/tele/SENSOR__
  * CLIMATE_TRANSCEIVER|TEMPERATURE_PATTERN __{{(parseJSON .).ZbReceived.THERMO1.Temperature}}__
  * CLIMATE_TRANSCEIVER|TEMPERATURE_EXTRACTOR __TEMPLATE__
  * CLIMATE_TRANSCEIVER|HUMIDITY_TOPIC __zbbridge/tele/SENSOR__
  * CLIMATE_TRANSCEIVER|HUMIDITY_PATTERN __{{(parseJSON .).ZbReceived.THERMO1.Humidity}}__
  * CLIMATE_TRANSCEIVER|HUMIDITY_EXTRACTOR __TEMPLATE__




