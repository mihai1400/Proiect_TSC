# Arhitectura Sistemului

## 1. Arhitectura Generală
Sistemul este construit în jurul **ESP32-C6-WROOM-1-N8** (microcontroler Wi-Fi 6 + Bluetooth 5 cu RISC-V) și include:

- **Alimentare & Baterie** (LDO, încărcător Li-Po, monitorizare nivel baterie)
- **Memorie** (Flash extern NOR 64MB, slot SD)
- **Senzori** (BME688 - temperatură/umiditate/presiune/VOC)
- **Afișaj** (E-Paper SPI)
- **Periferice** (RTC DS3231, butoane, LED-uri, USB-C)

## 2. Module & Interfețe cu ESP32-C6

### 2.1. Alimentare & Baterie
| Componentă                     | Interfață cu ESP32-C6       | Detalii                                  |
|--------------------------------|-----------------------------|------------------------------------------|
| LDO (XC6220A331MR-G)           | Nu (alimentare 3.3V)        | Reglează 3.3V pentru întreg sistemul     |
| Încărcător Baterie (MCP73831)  | GPIO4 (STAT)                | Semnalizează starea încărcării (LED)     |
| Monitor Baterie (MAX17048)     | I2C (SDA: GPIO8, SCL: GPIO9)| Măsoară nivelul bateriei (%/tensiune)    |

**Calcul consum:**
- ESP32-C6 (~80 mA în mod activ, ~5 µA în sleep)
- BME688 (~2.5 mA când active)
- E-Paper (~100 mA la refresh)
- **Total estimat:** ~200 mA (utilizare normală), sub 1 mA în sleep cu RTC activ

### 2.2. Memorie & Stocare
| Componentă            | Interfață cu ESP32-C6                          | Detalii                          |
|-----------------------|-----------------------------------------------|----------------------------------|
| Flash NOR (W25Q512JVEIQ) | SPI0 (CS: GPIO12, CLK: GPIO14, MOSI: GPIO13, MISO: GPIO11) | Stochează firmware/date |
| Slot SD (J4)          | SPI1 (CS: GPIO5, CLK: GPIO6, MOSI: GPIO7, MISO: GPIO10) | Stocare fișiere (FAT32) |

### 2.3. Senzori
| Componentă   | Interfață cu ESP32-C6       | Detalii                                  |
|--------------|-----------------------------|------------------------------------------|
| BME688       | I2C (SDA: GPIO8, SCL: GPIO9)| Senzor ambiental (temperatură, umiditate, presiune, VOC) |
| RTC (DS3231SN) | I2C (SDA: GPIO8, SCL: GPIO9)| Ceas precis (±2 ppm) cu alarmă           |

**Comunicație:**
- BME688 folosește I2C la 400 kHz (adresă `0x76`)
- DS3231 folosește I2C la 100 kHz (adresă `0x68`)

### 2.4. Afișaj E-Paper
| Componentă      | Interfață cu ESP32-C6                          | Detalii                          |
|-----------------|-----------------------------------------------|----------------------------------|
| Driver E-Paper  | SPI2 (CS: GPIO15, DC: GPIO16, RST: GPIO17, BUSY: GPIO18) | SPI dedicat pentru afișaj |

**Specificații:**
- Rezoluție: 800x600 (4.2" monocrom)
- Consum: ~100 mA la refresh, 0 mA în stare statică

### 2.5. Periferice & I/O
| Componentă   | Interfață cu ESP32-C6       | Detalii                                  |
|--------------|-----------------------------|------------------------------------------|
| Buton BOOT   | GPIO0                       | Resetare în mod flash                   |
| Buton CHANGE | GPIO1                       | Schimbă afișajul                        |
| LED-uri (D1-D12) | GPIO2, GPIO3            | Indicatori vizuali                      |
| USB-C (J1)   | GPIO20 (D+), GPIO21 (D-)    | Comunicație serială (CDC)               |

## 3. Alocarea Pinilor ESP32-C6
| Pin ESP32-C6 | Funcție       | Componentă            |
|--------------|---------------|-----------------------|
| GPIO0        | BOOT          | Buton BOOT            |
| GPIO1        | INPUT         | Buton CHANGE          |
| GPIO2-3      | OUTPUT        | LED-uri               |
| GPIO4        | INPUT         | MCP73831 (STAT)       |
| GPIO5        | CS            | Slot SD               |
| GPIO6-7      | SCK/MOSI      | Slot SD               |
| GPIO8-9      | SDA/SCL       | BME688 + DS3231 (I2C) |
| GPIO10       | MISO          | Slot SD               |
| GPIO11       | MISO          | Flash NOR             |
| GPIO12       | CS            | Flash NOR             |
| GPIO13-14    | MOSI/SCK      | Flash NOR             |
| GPIO15-18    | SPI EPD       | Afișaj E-Paper        |
| GPIO20-21    | USB D+/D-     | USB-C                 |

## 4. Optimizare Consum Energie
- **Sleep Mode:** ESP32-C6 intră în Deep Sleep cu RTC activ (~5 µA), trezit de DS3231 (alarmă) sau butoane
- **E-Paper:** Folosește energie doar la refresh
- **Senzori:** BME688 este pornit periodic (e.g., la 1 minut)

## 5. Diagramă
![Architecture Diagram](https://raw.githubusercontent.com/mihai1400/Proiect_TSC/main/diagrama.png)

## 6. BOM

| Device Module                       | Amount | Datasheet | Shop |
| ----------------------------------- | ------ | -------| --------|
| 112A-TAAR-R03_ATTEND                | 1      | https://www.attend.com.tw/en/product.php?act=view&id=253 | https://www.digikey.com/en/products/detail/attend-technology/112A-TAAR-R03/17633923 |
| BD5229G-TR                          | 1      | https://www.digikey.ph/en/products/detail/rohm-semiconductor/BD5229G-TR/658502 | https://www.digikey.ph/en/products/detail/rohm-semiconductor/BD5229G-TR/658502 |
| BUTTON_COSYOMV1                     | 3      | https://components101.com/switches/push-button | https://uk.farnell.com/wurth-elektronik/430182043816/switch-160gf-6x6mm-smd-4-3mm-act/dp/2065105?CMP=GRHB-OEMSECRETS101 |
| CPH3225A                            | 1      | https://www.digikey.com/en/products/detail/seiko-instruments/CPH3225A/8692444 | https://www.digikey.com/en/products/detail/seiko-instruments/CPH3225A/8692444 |
| DS3231SN                            | 1      | https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/DS3231SN/1197576 | https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/DS3231SN/1197576 |
| BME680                              | 1      | https://www.sparkfun.com/sparkfun-environmental-sensor-breakout-bme680-qwiic.html | https://www.sparkfun.com/sparkfun-environmental-sensor-breakout-bme680-qwiic.html |
| MCP73831                            | 1      | https://www.microchip.com/en-us/product/mcp73831 | https://www.microchipdirect.com/product/MCP73831T-2ADI/OT |
| C6-WROOM-1-N8                       | 1      | https://www.espressif.com/sites/default/files/documentation/esp32-c6-wroom-1_wroom-1u_datasheet_en.pdf | https://www.adafruit.com/product/5671 |
| FH34SRJ-24S-0.5SH_99_               | 1      | https://www.hirose.com/product/p/CL0580-1255-6-99 | https://www.hirose.com/product/p/CL0580-1255-6-99 |
| MAX17048G+T10                       | 1      | https://eu.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G+T10?qs=D7PJwyCwLAoGnnn8jEPRBQ%3D%3D | https://eu.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX17048G+T10?qs=D7PJwyCwLAoGnnn8jEPRBQ%3D%3D |
| MBR0530                             | 3      | https://www.digikey.com/en/products/detail/onsemi/MBR0530/1048783 | https://www.digikey.com/en/products/detail/onsemi/MBR0530/1048783 |
| PGB1010603MR                        | 6      | https://www.digikey.com/en/products/detail/littelfuse-inc/PGB1010603MR/715755 | https://www.digikey.com/en/products/detail/littelfuse-inc/PGB1010603MR/715755 |
| SAMACSYS_PARTS_USB4110-GF-A         | 1      | https://www.digikey.ro/en/products/detail/gct/USB4110-GF-A/10384547 | https://www.digikey.ro/en/products/detail/gct/USB4110-GF-A/10384547 |
| SI1308EDL-T1-GE3                    | 1      | https://www.digikey.com/en/products/detail/vishay-siliconix/SI1308EDL-T1-GE3/4876435 | https://www.digikey.com/en/products/detail/vishay-siliconix/SI1308EDL-T1-GE3/4876435 |
| SJ                                  | 1      | https://www.digikey.com/en/products/detail/cui-devices/SJ1-3514N/738685 | https://www.digikey.com/en/products/detail/cui-devices/SJ1-3514N/738685 |
| USBLC6-2SC6Y                        | 1      | https://www.digikey.com/en/products/detail/stmicroelectronics/USBLC6-2SC6Y/2819177 | https://www.digikey.com/en/products/detail/stmicroelectronics/USBLC6-2SC6Y/2819177 |
| W25Q512JVEIQ                        | 1      | https://www.digikey.com/en/products/detail/winbond-electronics/W25Q512JVEIQ/10244706 | https://www.digikey.com/en/products/detail/winbond-electronics/W25Q512JVEIQ/10244706 |
| XC6220A331MR-G                      | 1      | https://www.digikey.com/en/products/detail/torex-semiconductor-ltd/XC6220B331MR-G/2138177 | https://www.digikey.com/en/products/detail/torex-semiconductor-ltd/XC6220B331MR-G/2138177 |
| SD0805S020S1R0                      | 2      | https://www.digikey.com/en/products/detail/kyocera-avx/SD0805S020S1R0/3749517 | https://www.digikey.com/en/products/detail/kyocera-avx/SD0805S020S1R0/3749517 |
| LTSPICE_C 100nF                     | 8      | https://www.digikey.com/en/products/detail/nextgen-components/0402B104K500HI/22601924 | https://www.digikey.com/en/products/detail/nextgen-components/0402B104K500HI/22601924 |
| LTSPICE_C 10nF                      | 1      | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 |
| LTSPICE_C 4.7uF                     | 6      | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 |
| LTSPICE_C 1uF                       | 10     | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 | https://www.amazon.com/ALLECIN-Electrolytic-Capacitor-0-2x0-43in-Capacitors/dp/B0CMQC587X/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1 |
| LTSPICE_C 0.1uF                     | 1      | https://www.digikey.com/en/products/detail/samsung-electro-mechanics/CL10E104KC8VPNC/20498486 | https://www.digikey.com/en/products/detail/samsung-electro-mechanics/CL10E104KC8VPNC/20498486 |
| ADAFRUIT_LED                        | 1      | https://www.adafruit.com/product/4684 | https://www.adafruit.com/product/4684 |
| ESP32_VARISTOR                      | 1      | https://uk.rs-online.com/web/p/varistors/9012743 | https://uk.rs-online.com/web/p/varistors/9012743 |
| CT3528 (RCL_CPOL-EU) 100uF          | 1      | https://www.datasheetarchive.com/?q=ct3528 |  |
| DMG2305UX-7 20V/4.2A/52mO/1.4W      | 2      | https://www.digikey.com/en/products/detail/diodes-incorporated/DMG2305UX-7/4340667 | https://www.digikey.com/en/products/detail/diodes-incorporated/DMG2305UX-7/4340667 |
| IND-4828-WE-TPC-WRE                 | 1      | https://www.axel-gl.com/en/asone/d/64-0172-48/ | https://www.axel-gl.com/en/asone/d/64-0172-48/ |
| JS-1MM(QWIIC_CONNECTOR)             | 1      | https://www.sparkfun.com/qwiic-jst-connector-smd-4-pin-horizontal.html | https://www.sparkfun.com/qwiic-jst-connector-smd-4-pin-horizontal.html |
| LTSPICE_R 5k1                       | 2      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 100k                      | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 2.2                       | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 10k                       | 16     |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 0.47                      | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 2k                        | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 200                       | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| LTSPICE_R 15                        | 1      |  | https://www.amazon.com/BOJACK-Values-Resistor-Resistors-Assortment/dp/B08FD1XVL6/ref=sr_1_1_sspa?sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY |
| Test Pad TP20R                      | 17     | https://us.misumi-ec.com/vona2/detail/221005496391/?HissuCode=TP20R-24 | https://us.misumi-ec.com/vona2/detail/221005496391/?HissuCode=TP20R-24 |


## 7. Concluzii
- ESP32-C6 este nucleul sistemului, cu 3 interfețe SPI (Flash, SD, E-Paper) și I2C partajat (BME688 + DS3231)
- BME688 oferă date ambientale, iar DS3231 menține timpul precis
- Consum redus în sleep (~5 µA) permite funcționare pe baterie >1 lună

Probleme : 
 - Nu pot genera fiserul .step (se blocheaza Fusion-ul cand incerc)
