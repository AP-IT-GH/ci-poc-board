### TODO

[x] Power Supply Connections
- [x] External Reset
- [ ] Clock & Oscillator
- [x] Programming & debug connectie

- [ ] 4x UART
- [ ] 1x SPI
- [ ] 1x I²C
- [ ] Connectie batterijniveau voor ADC
- [ ] Connectie Salinity sensor



#### Voeding

Gebruiken we een [EEMB LP103395 batterij met een AP7343D-33W5-7 als 3.3V regelaar](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/#/./Pages/Analyse/Hardware?id=voeding) of een [Li-Po 2 Cell batterij en een LD1117 als 3.3V regelaar](https://github.com/AP-Project-Analysis-2IT-IOT-1TVT-IOT/Daan-Olivier/blob/main/Blueprint.md#onderliggende-argumentatie)?

EEMB LP103395: [Datasheet](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/Pages/Apendix/Datasheets/EEMB_LP103395.pdf), [Documentatie fabrikant](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/#/./Pages/Analyse/Hardware?id=voeding), [Winkel](https://www.amazon.com/EEMB-3700mAh-Rechargeable-Connector-Certified/dp/B08215B4KK)

AP7343D-33W5-7: [Datasheet](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/Pages/Apendix/Datasheets/AP7343D.pdf), [Documentatie fabrikant](https://ap-project-analysis-2it-iot-1tvt-iot.github.io/bavo-birk-docs-luchtsensor/#/./Pages/Analyse/Hardware?id=voeding), [Winkel](https://eu.mouser.com/ProductDetail/Diodes-Incorporated/AP7343D-33W5-7?qs=EDJ299gKAZQ2xV9YzJ9Zog%3D%3D)

Li-Po 2 Cell: [Winkel](https://www.conrad.be/p/conrad-energy-lipo-accupack-74-v-2400-mah-aantal-cellen-2-20-c-softcase-xt60-1344133?WT.srch=1&gclid=CjwKCAiAh_GNBhAHEiwAjOh3ZODyQpj1PCOdHiGXfnYxeG0l__VZOiLFqiP5MSZwps0pyi__jmN_WhoC9LsQAvD_BwE&gclsrc=aw.ds&insert=8J&t=1&tid=13894944235_122657379817_pla-301443522443_pla-1344133&utm_campaign=shopping-feed&utm_content=free-google-shopping-clicks&utm_medium=surfaces&utm_source=google&utm_term=1344133&vat=true), [Datasheet](https://asset.conrad.com/media10/add/160267/c1/-/en/001344133SD01/veiligheidsvoorschriften-1344133-conrad-energy-lipo-accupack-74-v-2400-mah-aantal-cellen-2-20-c-softcase-xt60.pdf)

LD1117: [Winkel](https://www.conrad.be/p/stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a-1184973?searchTerm=LD1117&searchType=suggest&searchSuggest=product), [Datasheet](https://asset.conrad.com/media10/add/160267/c1/-/en/001184973DS01/datablad-1184973-stmicroelectronics-ld1117av33-spanningsregelaar-lineair-to-220ab-positief-vast-1-a.pdf)



#### AT SAMD 21 Schematic Checklist

Datasheet p1108-p1118

##### Power Supply Connections

De MCU heeft twee VDDIO pinnen pin17 en pin36, deze zouden intern doorverbonden zijn dus enkel pin17 is momenteel aangesloten.

##### External Analog Reference Connections

Niet nodig want we gebruiken (nog) geen external analog references.

##### External Reset Circuit

EFT = electrical fast transients -> vonk bij het indrukken van een knop

Wij hebben deze EFT Immunity Enhancement niet nodig dus gebruiken "Figure 45-4. External Reset Circuit Schematic".

##### Clocks and Crystal Oscillators

https://blog.thea.codes/understanding-the-sam-d21-clocks/

We gebruiken een externe 32,768kHz klok en zullen via de clock multiplier deze frequentie verhogen.

<u>Specifieke crystal oscillator en footprint nog bevestigen.</u>

Zou het gemakkelijker en of zuiniger zijn om enkel de interne 8MHz klok te gebruiken? Moeten we voor de SERCOM poorten sowieso nog een andere klok gebruiken dan de systeemklok of <u>kan alles volledig werken met 8MHz of 48MHz?</u>

> In many cases you can use the ultra low power internal oscillator (`OSCULP32K`) and save a lot on power consumption. - [Understanding the SAM D21 clocks](https://blog.thea.codes/understanding-the-sam-d21-clocks/)

In bovenstaand geval moeten we geen externe oscillator en condensatoren aan te sluiten en besparen we ruimte op de PCB en geld.

##### Programming and Debug Ports

Pull-up weerstand op SWCLK!

[Dit bord](https://www.kickstarter.com/projects/hamishmorley/minima-accelerate-your-ideas-into-products) heeft een RX en TX pin op de SWCLK en SWDIO pinnen. Hoe werkt dat?

##### USB Interface

De USB interface zit op PA23, PA24 en PA25 dus bij gebruik van USB zou SERCOM3 wegvallen.

Het zou ook mogelijk zijn om (met de juiste bootloader) de USB aansluiting te gebruiken als seriële interface. ([bron](https://www.avdweb.nl/arduino/samd21/sam-d21))

Waarschijnlijk hebben we deze USB interface niet rechtstreeks nodig. Maken we gebruik van een programmer chip zodat we -zoals in het vorige project- een USB-C aansluiting hebben om langs te programmeren en communiceren? SWD heeft geen RX en TX verbinding dus hoe geven we info weer op de seriële monitor? Moeten we daarvoor een SERCOM opgeven? Of verbinden we altijd een debugger als we willen programmeren en communiceren?



#### SERCOM Aansluiten

Iedere SERCOM pin van de SAM D21 heeft een `SERCOMx_PADn` label gekregen voor gemakkelijke referentie. Er werd gekeken naar de C kolom over SERCOM in Table 7-1. 

<u>Wat is SERCOM-ALT (kolom D)?</u> [ref1](https://microchipsupport.force.com/s/article/SERCOM-muxing-on-SAM-D-L-C-devices), [ref2<u>](https://learn.adafruit.com/using-atsamd21-sercom-to-add-more-spi-i2c-serial-ports/muxing-it-up)</u>

<u>Kunnen we met de ATSAMD21**G**18A wel gebruik maken van de 6 SERCOM aansluitingen?</u> Hebben we niet een D21**J** nodig om genoeg pinnen ter beschikking te hebben? Zoals ik het las heeft de D21G wel 6 SERCOM's maar kunnen er daarvan via de pinnen maar 4 tegelijkertijd gebruikt worden.

>E = 32 Pins
>G = 48 Pins
>J = 64 Pins



SERCOM3_PAD2 en _PAD3 moeten in gebruik zijn of een pull-up weerstand krijgen (dat zijn de USB pinnen). 

> If the PA24 and PA25 pins are not connected, it is recommended to enable a pull-up on PA24 and PA25 through input GPIO mode. The aim is to avoid an eventually extract power consumption (<1mA) due to a not stable level on pad. The port PA24 and PA25 doesn't have Drive Strength option.

[ATSAMD D21 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/SAM-D21DA1-Family-Data-Sheet-DS40001882G.pdf)

