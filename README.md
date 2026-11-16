# ESP8266 CO2-sensor (ESPHome)

Een zelfgebouwde CO2-meter voor in huis: een NodeMCU (ESP8266) met een MH-Z19B CO2-sensor,
een BME280 voor temperatuur, luchtvochtigheid en luchtdruk, en een klein OLED-scherm. Hij
draait [ESPHome](https://esphome.io) en meldt zich daarmee direct in
[Home Assistant](https://www.home-assistant.io), zonder MQTT-broker ertussen.

*English summary: a DIY indoor CO2 meter (NodeMCU ESP8266 + MH-Z19B + BME280 + 0.96" OLED),
originally built with ESP Easy after the Duurzaam Meerhoven design by Stefan van der Eijk,
now running ESPHome with native Home Assistant integration.*

## Herkomst en credits

Deze meter is gebouwd naar het ontwerp en de bouwhandleiding van **Stefan van der Eijk**,
gemaakt voor het buurtinitiatief **Duurzaam Meerhoven** (Eindhoven), rond 2019:

- De behuizing en het ontwerp: [CO2, temperature & humidity sensor door stefan_van_der_eijk
  op Thingiverse (thing 3465447)](https://www.thingiverse.com/thing:3465447). De STL-bestanden
  staan daar en zijn bewust niet in deze repo gekopieerd.
- De bouwhandleiding (een Google Doc van Stefan) stond op
  `onsplatform.tv/duurzaammeerhoven/zelf-een-co2-meter-bouwen`; een archiefkopie:
  [web.archive.org](https://web.archive.org/web/20201101002426/https://www.onsplatform.tv/duurzaammeerhoven/zelf-een-co2-meter-bouwen).
- Het hoofdstuk over ThingSpeak in die handleiding is ondertekend door "Lars"; dat is
  vermoedelijk **Lars Boelen**, die er in juni 2019
  [naar verwees](https://twitter.com/larsboelen/statuses/1144857952916054016).
  ThingSpeak wordt in deze ESPHome-versie niet gebruikt.
- De oorspronkelijke firmware was [ESP Easy](https://github.com/letscontrolit/ESPEasy)
  (plugins MH-Z19, BME280 en OLED). De handleiding verwees voor ESPHome naar de
  [ESPHome-workshop van Galagaking](https://galagaking.org/2019/11/05/esphome-workshop).

Voor zover bekend is er geen eerdere Git-repo waarop het ontwerp is gebaseerd. Deze repo
bevat alleen de eigen ESPHome-omzetting; de rechten op de handleiding, foto's en de behuizing
liggen bij de makers hierboven.

## Hardware

| Onderdeel | Aansluiting |
|---|---|
| NodeMCU v1.0 (ESP8266, CP2102) | voeding 5 V via micro-USB |
| MH-Z19B CO2-sensor | Vin → 5 V (VIN), GND; sensor-TX → GPIO4 (D2), sensor-RX → GPIO5 (D1) |
| BME280 (I2C, adres 0x76) | 3V3, GND; SDA → GPIO12 (D6), SCL → GPIO13 (D7) |
| 0,96" OLED 128×64 (SH1106-controller, I2C 0x3C) | 3V3, GND; zelfde I2C-bus |

Let op: veel van deze goedkope 0,96"-schermpjes hebben een SH1106-controller en geen
SSD1306, ook als de winkel SSD1306 zegt. Dit exemplaar is SH1106 en staat 180° gedraaid.

## ESPHome

Entiteiten in Home Assistant (apparaat "CO2-sensor woonkamer"): CO2, Temperatuur,
Luchtvochtigheid, Luchtdruk, Status, Wifi-signaal, Uptime, de knoppen "Herstarten" en
"Kalibreer CO2 op 400 ppm", de schakelaar "CO2 automatische basiskalibratie" en
"Temperatuur MH-Z19 (intern)" (standaard uitgeschakeld). Het scherm toont elke 5 seconden om
de beurt temperatuur, luchtvochtigheid en CO2.

- `co2-sensor-woonkamer.yaml`: de configuratie.
- `secrets.example.yaml`: kopieer naar `secrets.yaml` en vul wifi, API-sleutel en
  OTA-wachtwoord in. `secrets.yaml` staat in `.gitignore`.

Eerste keer flashen via USB:

```
esphome run co2-sensor-woonkamer.yaml --device COM6
```

Lukt dat niet (bij dit bordje gaf de CP2102 op 460800 baud leesfouten), flash dan het
gecompileerde bestand op 115200 baud:

```
esphome compile co2-sensor-woonkamer.yaml
esptool --port COM6 --baud 115200 write-flash --erase-all 0x0 .esphome/build/co2-sensor-woonkamer/.pioenvs/co2-sensor-woonkamer/firmware.bin
```

Daarna kan het draadloos (OTA). Home Assistant vindt het apparaat vanzelf via de
ESPHome-integratie.

### Kalibratie

De MH-Z19B gebruikt automatische basiskalibratie (ABC): hij neemt aan dat hij minstens één keer
per 24 uur buitenlucht (ongeveer 420 ppm) ziet. Staat hij in een ruimte die nooit goed
gelucht wordt, zet ABC dan uit en kalibreer handmatig: zet de sensor 20 minuten buiten of
bij een open raam en druk in Home Assistant op de knop "Kalibreer CO2 op 400 ppm".
ABC staat standaard aan en is in Home Assistant uit te zetten met de schakelaar
"CO2 automatische basiskalibratie".

## Terug naar ESP Easy

Een volledige 4 MB-flashback-up van de oude ESP Easy-installatie (mega-20250430) kan worden
teruggezet met:

```
esptool --port COM6 --baud 115200 write-flash 0x0 espeasy-full-4MB-2026-11-16.bin
```

(De back-up zelf staat niet in deze repo: hij bevat wifi-gegevens.)
