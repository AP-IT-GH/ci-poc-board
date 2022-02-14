### TODO

- [x] Power Supply Connections
- [ ] Spanningsregelaars 3.3V en 5V

- [x] External Reset
- [ ] Clock & Oscillator
- [x] Programming & debug connectie
- [x] 4x UART
- [x] 1x SPI
- [x] 1x I²C
- [ ] Connectie batterijniveau voor ADC
- [ ] Connectie Salinity sensor
- [ ] RN2483 LoRaWAN module



#### Voeding

We gebruiken een [EEMB LP103395 batterij](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/#/./Pages/Analyse/Hardware?id=voeding) en een [LD1117 als 3.3V en 5V regelaar](https://github.com/AP-Project-Analysis-2IT-IOT-1TVT-IOT/Daan-Olivier/blob/main/Blueprint.md#onderliggende-argumentatie).

EEMB LP103395: [Datasheet](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/Pages/Apendix/Datasheets/EEMB_LP103395.pdf), [Documentatie fabrikant](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/#/./Pages/Analyse/Hardware?id=voeding), [Winkel](https://www.amazon.com/EEMB-3700mAh-Rechargeable-Connector-Certified/dp/B08215B4KK)

LD1117: [Winkel](https://www.conrad.be/p/stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a-1184973?searchTerm=LD1117&searchType=suggest&searchSuggest=product), [Datasheet](https://asset.conrad.com/media10/add/160267/c1/-/en/001184973DS01/datablad-1184973-stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a.pdf)



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

- [ ] <u>Checken of `XIN32` en `XOUT32` niet in conflixt staan met PA00 en PA01 van USART_1.</u> 



##### Programming and Debug Ports

Pull-up weerstand op SWCLK!

We gebruiken de "Figure 45-12. 10-pin JTAGICE3 Compatible Serial Wire Debug Interface".



##### USB Interface

We gebruiken geen USB Interface.



#### SERCOM Aansluiten

De gebruikte SERCOM pinnen van de SAM D21 heeft een `SERCOMx_PADn_y` label gekregen voor gemakkelijke referentie. Er werd gekeken naar de lay-out die Atmel START gaf.

SERCOM-ALT [ref1](https://microchipsupport.force.com/s/article/SERCOM-muxing-on-SAM-D-L-C-devices), [ref2](https://learn.adafruit.com/using-atsamd21-sercom-to-add-more-spi-i2c-serial-ports/muxing-it-up)

| Protocol | SERCOM  | Pinnen           | Functie         |
| -------- | ------- | ---------------- | --------------- |
| USART_0  | SERCOM0 | PA04, PA05       | TX, RX          |
| USART_1  | SERCOM1 | PA00, PA01       | TX, RX          |
| USART_2  | SERCOM2 | PA08, PA09       | TX, RX          |
| USART_3  | SERCOM3 | PA16, PA17       | TX, RX          |
| I2C_0    | SERCOM4 | PA12, PA13       | SDA, SCL        |
| SPI_0    | SERCOM5 | PA20, PA22, PA23 | MISO, MOSI, SCK |



> If the PA24 and PA25 pins are not connected, it is recommended to enable a pull-up on PA24 and PA25 through input GPIO mode. The aim is to avoid an eventually extract power consumption (<1mA) due to a not stable level on pad. The port PA24 and PA25 doesn't have Drive Strength option. (dat zijn de USB pinnen). 

[ATSAMD D21 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/SAM-D21DA1-Family-Data-Sheet-DS40001882G.pdf)

