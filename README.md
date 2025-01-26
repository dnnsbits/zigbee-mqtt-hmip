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
 
    `{"ZbReceived":{"0x5480":{"Device":"0x5480","Temperature":20.3,"Endpoint":1,"LinkQuality":76}}}`
    
    wird zu
    
    `{"ZbReceived":{"THERMO1":{"Device":"0x5480","Name":"THERMO1","Temperature":20.9,"Endpoint":1,"LinkQuality":76}}}`
 
* check im MQTT Explorer
  * Telemetriedaten Topic __zbbridge/tele__
    * __STATE__ über die Bridge
    * __SENSOR__ über verbundene Sensoren
   
## Geräte

### ZB Bridge

#### Geräte benennen

* Gerät benennen -> Werkzeuge/Konsole: __ZbName 0x5480,NAME01__

### Sonoff SNZB-02D Temperatur- und Luftfeuchtigkeitssensor mit Display

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

### Nous Zigbee Socket A1Z (Plug) Zwischenstecker Schaltsteckdose

#### MQTT Topic 

Topic _zbbridge/tele/SENSOR_ Value ist _OHNE_ Leerzeichen und Umbrüche!

    {
      "ZbReceived": {
        "PLUG1": {
          "Device": "0x1234",
          "Name": "PLUG1",
          "RMSVoltage": 231,
          "RMSCurrent": 0,
          "ActivePower": 0,
          "CurrentSummationDelivered": "0x00000000000D",
          "Endpoint": 1,
          "LinkQuality": 98
        }
      }
    }

#### ccu-jack

* Virtuelle Geräte/Erstellen
  * Gerätesymbol __HM-ES-PMSw1-Pl__
  * Kanal 1: __MQTT Schaltaktor mit Rückmeldung__
  * Kanal 2: __MQTT Energiemessung__
 
#### CCU3

!alle Playloads und Topics OHNE Leerzeichen

* Geräte Einstellungen (für Kanal 1 - Schaltaktor)
  * SWITCH|COMMAND_TOPIC __zbbridge/cmnd/ZbSend__
  * SWITCH|ON_PAYLOAD __{"device":"PLUG1","send":{"Power":"1"}}__
  * SWITCH|OFF_PAYLOAD __{"device":"PLUG1","send":{"Power":"0"}}__
  * SWITCH|FEEDBACK_TOPIC __zbbridge/tele/SENSOR__
  * SWITCH|ON_PATTERN __"Name":"PLUG1","Power":1__
  * SWITCH|OFF_PATTERN __"Name":"PLUG1","Power":0__
  * SWITCH|MATCHER __CONTAINS__
* Geräte Einstellungen (für Kanal 2 - Messwert-Kanal)
  * POWERMETER|POWER_TOPIC __zbbridge/tele/SENSOR__
  * POWERMETER|POWER_PATTERN __{{(parseJSON .).ZbReceived.PLUG1.ActivePower}}__
  * POWERMETER|POWER_EXTRACTOR __TEMPLATE__
  * POWERMETER|CURRENT_TOPIC __zbbridge/tele/SENSOR__
  * POWERMETER|CURRENT_PATTERN __{{(parseJSON .).ZbReceived.PLUG1.RMSCurrent}}__
  * POWERMETER|CURRENT_EXTRACTOR __TEMPLATE__
  * POWERMETER|VOLTAGE_TOPIC __zbbridge/tele/SENSOR__
  * POWERMETER|VOLTAGE_PATTERN __{{(parseJSON .).ZbReceived.PLUG1.RMSVoltage}}__
  * POWERMETER|VOLTAGE_EXTRACTOR __TEMPLATE__





