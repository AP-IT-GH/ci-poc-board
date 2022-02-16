### TODO Schema

- [x] Power Supply Connections

- [x] Spanningsregelaars 3.3V en 5V

- [x] External Reset

- [x] Clock & Oscillator

- [x] Programming & debug connectie

- [x] 4x UART

- [x] 1x SPI

- [x] 1x I²C

- [x] Connectie batterijniveau voor ADC

- [x] Connectie Salinity sensor

- [x] RN2483 LoRaWAN module

#### Voeding

We gebruiken een [Li-Po 2 Cell betterij](https://github.com/AP-Project-Analysis-2IT-IOT-1TVT-IOT/Daan-Olivier/blob/main/Blueprint.md#onderliggende-argumentatie) en een [LD1117 als 3.3V en 5V regelaar](https://github.com/AP-Project-Analysis-2IT-IOT-1TVT-IOT/Daan-Olivier/blob/main/Blueprint.md#onderliggende-argumentatie).

Li-Po 2 Cell: [Winkel](https://www.conrad.be/p/conrad-energy-lipo-accupack-74-v-2400-mah-aantal-cellen-2-20-c-softcase-xt60-1344133?WT.srch=1&gclid=CjwKCAiAh_GNBhAHEiwAjOh3ZODyQpj1PCOdHiGXfnYxeG0l__VZOiLFqiP5MSZwps0pyi__jmN_WhoC9LsQAvD_BwE&gclsrc=aw.ds&insert=8J&t=1&tid=13894944235_122657379817_pla-301443522443_pla-1344133&utm_campaign=shopping-feed&utm_content=free-google-shopping-clicks&utm_medium=surfaces&utm_source=google&utm_term=1344133&vat=true), [Datasheet](https://asset.conrad.com/media10/add/160267/c1/-/en/001344133SD01/veiligheidsvoorschriften-1344133-conrad-energy-lipo-accupack-74-v-2400-mah-aantal-cellen-2-20-c-softcase-xt60.pdf)

LD1117: [Winkel](https://www.conrad.be/p/stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a-1184973?searchTerm=LD1117&searchType=suggest&searchSuggest=product), [Datasheet](https://asset.conrad.com/media10/add/160267/c1/-/en/001184973DS01/datablad-1184973-stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a.pdf)



De LD1117 chips zijn aangesloten volgens "Figure 3. Application circuit (for fixed output voltages)". 1x 5V en 1x 3.3V. 

De dropout voltage van deze regulator is 1.05V als we rekenen op een belasting van 500mAh. Om een spanning van 5V te kunnen leveren zal de regulator dus een input voltage moeten hebben van 5V + 1.05V = 6.05V. Onze batterijspanning zal dus boven 6.05V moeten blijven.

ADC spanningsdeler voor batterij:

R1 = 10 kOhm; R2 = 4.7 kOhm



#### AT SAMD 21 Schematic Checklist

Datasheet p1108-p1118

##### Power Supply Connections

- [ ] L1 10µH footprint nakijken.

De MCU heeft twee VDDIO pinnen pin17 en pin36, deze zijn intern doorverbonden zijn dus enkel pin17 is momenteel aangesloten.

##### External Analog Reference Connections

Niet nodig want we gebruiken (nog) geen external analog references.

##### External Reset Circuit

EFT = electrical fast transients -> vonk bij het indrukken van een knop

Wij hebben deze EFT Immunity Enhancement niet nodig dus gebruiken "Figure 45-4. External Reset Circuit Schematic".

##### Clocks and Crystal Oscillators

[Understanding the SAM D21 clocks](https://blog.thea.codes/understanding-the-sam-d21-clocks/)

We maken de mogelijkheid voor een externe 32,768kHz klok, deze hoeft op de PCB niet perse bestukt worden.

- [Capacitor](https://be.farnell.com/multicomp/mc0805n180j500ct/cap-18pf-50v-5-c0g-np0-0805/dp/1759194): MC0805N180J500CT, 18 pF, 0805
- [Crystal](https://be.farnell.com/fox-electronics/fx135a-327/crystal-32-768khz-12-5pf-smd/dp/2064037): FX135A-327, 32.768 kHz, SMD

Aangesloten volgens: "Figure 45-9. External Real Time Oscillator with Load Capacitor".

`XIN32` en `XOUT32` zitten op de enige beschikbare aansluiting voor SERCOM1 (PA00 en PA01) dus SERCOM1 zal niet gebruikt worden.

##### Programming and Debug Ports

Pull-up weerstand op SWCLK!

We gebruiken de "Figure 45-12. 10-pin JTAGICE3 Compatible Serial Wire Debug Interface".

##### USB Interface

We gebruiken geen USB Interface.

#### SERCOM Aansluiten

De gebruikte SERCOM pinnen van de SAM D21 heeft een `SERCOMx_PADn_y` label gekregen voor gemakkelijke referentie. Er werd gekeken naar de lay-out die Atmel START gaf.

SERCOM-ALT [ref1](https://microchipsupport.force.com/s/article/SERCOM-muxing-on-SAM-D-L-C-devices), [ref2](https://learn.adafruit.com/using-atsamd21-sercom-to-add-more-spi-i2c-serial-ports/muxing-it-up)

| Gebruik | SERCOM  | Pinnen           | Functie                                                     |
| ------- | ------- | ---------------- | ----------------------------------------------------------- |
| RN2384  | SERCOM0 | PA04, PA05, PA07 | TX, RX, <span style="text-decoration:overline">RESET</span> |
| USART_1 | SERCOM5 | PA22, PA23       | TX, RX                                                      |
| USART_2 | SERCOM2 | PA08, PA09       | TX, RX                                                      |
| USART_3 | SERCOM3 | PA16, PA17       | TX, RX                                                      |
| I2C_0   | SERCOM4 | PA12, PA13       | SDA, SCL                                                    |

> If the PA24 and PA25 pins are not connected, it is recommended to enable a pull-up on PA24 and PA25 through input GPIO mode. The aim is to avoid an eventually extract power consumption (<1mA) due to a not stable level on pad. The port PA24 and PA25 doesn't have Drive Strength option. (dat zijn de USB pinnen). 

[ATSAMD D21 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/SAM-D21DA1-Family-Data-Sheet-DS40001882G.pdf)

#### RN2483 LoRaWAN module

[RN2483 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/RN2483-Low-Power-Long-Range-LoRa-Technology-Transceiver-Module-DS50002346F.pdf) 

Symbool van AirQualitySensor gebruikt.

`PA6` die verbonden is met `RN2384_RESET` is altijd HIGH. De RN2483 module is te resetten door deze pin even op LOW te zetten. 

<u>Gebruiken we `UART_CTS` en `UART_RTS` als handshake mechanisme?</u> 

<u>Sluiten we Internal Program Pins aan met een 6-pin ICSP header op onze PCB?</u>

We gebruiken RFH voor 868MHz communicatie. 

> When routing RF paths, use proper strip lines with an impedance of 50 Ohm.

De GPIO pinnen van deze module gebruiken we niet. 



### Salinity Sensor

[Ref1](https://me121.mme.pdx.edu/lib/exe/fetch.php?media=lecture:salinity_measurements_with_arduino_slides.pdf), [Ref2](https://www.teachengineering.org/activities/view/nyu_probe_activity1), [Lib1](https://www.arduino.cc/reference/en/libraries/conductivitylib/)

De zoutsensor bestaat uit twee elektrodes die op een bepaalde afstand van elkaar liggen. Onder water zal, afhankelijk van de hoeveelheid zout er meer of minder stroom kunnen vloeien. Bij een constante stroom door de elektrodes zullen ze sneller verslijten dus zullen we de polariteit wisselen tussen twee digitale pinnen.




