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

| Reference      | X     | Y      | Rotation | Value                                      | Footprint                                  |
|---------------|-------|--------|----------|--------------------------------------------|-------------------------------------------|
| BOOT_BUTTON   | 36.65 | 112.67 | 180.00   | BUTTON_CUSYOMV1                            | MYBUTTON                                  |
| C1            | 170.35 | 11.74  | 90.00    | 100nf                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C10           | 72.51 | 112.21 | 270.00   | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C10_SUPERCAP  | 110.76 | 48.41  | 270.00   | CPH3225A                                   | CAPCP3225X100N                            |
| C1_BAT        | 126.31 | 46.88  | 0.00     | 4.7uF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C1_BAT1       | 139.51 | 50.62  | 90.00    | 4.7uF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C1_BAT2       | 137.87 | 61.45  | 90.00    | 4.7uF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C2            | 171.77 | 11.84  | 90.00    | 100nf                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C2_BAT        | 124.25 | 59.41  | 0.00     | 4.7uF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C3            | 136.08 | 50.48  | 270.00   | 100uF TANT                                 | RCL_CT3528                                |
| C4            | 120.74 | 34.42  | 90.00    | 4.7uF/25V                                  | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C4_USB        | 163.22 | 50.13  | 0.00     | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C5            | 57.57 | 112.21 | 270.00   | 1uF                                        | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C5_USB        | 163.21 | 51.43  | 0.00     | 4.7uF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C6            | 32.61 | 112.21 | 270.00   | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C7            | 102.62 | 28.14  | 0.00     | 10uF                                       | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C8            | 103.27 | 45.19  | 0.00     | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| C9            | 22.45 | 104.72 | 90.00    | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| CHANGE_BUTTON | 76.61 | 112.73 | 180.00   | BUTTON_CUSYOMV1                            | MYBUTTON                                  |
| C_DELAY       | 123.20 | 50.60  | 90.00    | 100nF                                      | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| D1            | 162.18 | 54.30  | 0.00     | USBLC6-2SC6Y                               | SOT95P280X145-6N                          |
| D10           | 68.35 | 36.39  | 90.00    | PGB1010603MR                               | DIOC1608X36N                              |
| D11           | 27.59 | 102.84 | 0.00     | PGB1010603MR                               | DIOC1608X36N                              |
| D12           | 93.99 | 24.23  | 0.00     | PGB1010603MR                               | DIOC1608X36N                              |
| D2            | 125.60 | 50.52  | 90.00    | ESP32_WROVER_AVX---SD0805S020S1R0_AVX_SD0805S020S1R0_0_0AVX_SD0805S020S1R0_0_0 | ESP32_WROVER_AVX---SD0805S020S1R0_AVX_SD0805S020S1R0_0 |
| D3            | 116.94 | 33.52  | 180.00   | MBR0530                                    | SOD3716X135N                              |
| D4            | 116.96 | 29.97  | 0.00     | MBR0530                                    | SOD3716X135N                              |
| D5            | 103.34 | 36.94  | 180.00   | MBR0530                                    | SOD3716X135N                              |
| D6            | 50.32 | 49.59  | 90.00    | PGB1010603MR                               | DIOC1608X36N                              |
| D7            | 104.87 | 60.98  | 0.00     | ESP32_WROVER_AVX---SD0805S020S1R0_AVX_SD0805S020S1R0_0_0AVX_SD0805S020S1R0_0_0 | ESP32_WROVER_AVX---SD0805S020S1R0_AVX_SD0805S020S1R0_0 |
| D8            | 63.33 | 36.29  | 90.00    | PGB1010603MR                               | DIOC1608X36N                              |
| D9            | 66.14 | 36.39  | 90.00    | PGB1010603MR                               | DIOC1608X36N                              |
| EPC_C6        | 110.16 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPC_C8        | 106.34 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C1        | 88.53 | 16.59  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C10       | 102.52 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C11       | 100.61 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C12       | 98.70 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C2        | 89.69 | 21.78  | 0.00     | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C5        | 124.54 | 23.39  | 90.00    | 0.1uF/50V                                  | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C7        | 108.15 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| EPD_C9        | 104.53 | 24.09  | 90.00    | 1uF/50V                                    | ESP32_WROVER_EAGLE-LTSPICE_C0402          |
| GHC_LED       | 127.92 | 59.53  | 90.00    |                                            | ADAFRUIT_CHIP-LED0603                     |
| IC1           | 129.58 | 50.48  | 0.00     | BD5229G-TR                                 | SOT95P280X125-5N                          |
| IC4           | 61.69 | 106.50 | 0.00     | XC6220A331MR-G                             | SOT95P280X120-5N                          |
| J1            | 102.58 | 17.78  | 0.00     | FH34SRJ-24S-0.5SH_99_                      | FH34SRJ24S05SH99                          |
| J2            | 179.27 | 58.60  | 90.00    | SAMACSYS_PARTS_USB4110-GF-A                | SAMACSYS_PARTS_USB4110GFA                 |
| J3            | 179.74 | 23.56  | 90.00    | QWIIC_RIGHT_ANGLE                          | JST04_1MM_RA                              |
| J4            | 21.00 | 93.15  | 270.00   | 112A-TAAR-R03_ATTEND                       | 112ATAARR03ATTEND                         |
| L1            | 105.23 | 33.55  | 0.00     | 744043680IND_4828-WE-TPC_WRE               | IND_4828-WE-TPC_WRE                       |
| MCP73831      | 132.99 | 58.63  | 90.00    | ESP32_WROVER_SPARKFUN-IC-POWER_MCP73831    | ESP32_WROVER_SPARKFUN-IC-POWER_SOT23-5    |
| PFMF.050.1    | 161.86 | 47.20  | 180.00   | ESP32C6_VARISTORCN1812                     | ESP32C6_VARISTOR_CT/CN1812                |
| Q1            | 140.29 | 56.91  | 0.00     | 20V/4.2A/52mΩ/1.4W                         | ESP32_WROVER_SPARKFUN-DISCRETESEMI_SOT23-3|
| Q2            | 99.53 | 29.04  | 270.00   | 20V/4.2A/52mΩ/1.4W                         | ESP32_WROVER_SPARKFUN-DISCRETESEMI_SOT23-3|
| Q3            | 84.96 | 23.36  | 90.00    | SI1308EDL-T1-GE3                           | SOT65P210X110-3N                          |
| R1            | 31.93 | 45.77  | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R10           | 24.40 | 104.69 | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R1_BAT        | 127.93 | 57.90  | 0.00     | 200                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R1_PINH       | 167.34 | 11.74  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R1_PINH1      | 107.99 | 48.29  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R1_PWRUSB     | 105.28 | 59.16  | 180.00   |                                            | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2            | 86.10 | 18.23  | 0.00     | 2.2                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2-PINH       | 168.83 | 11.74  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2-PINH1      | 108.73 | 58.21  | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2-USB        | 157.78 | 54.29  | 90.00    | 5k1                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2-USB1       | 165.38 | 56.78  | 90.00    | 5k1                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R2_BAT        | 132.01 | 54.65  | 90.00    | 2K                                         | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R3            | 88.94 | 24.31  | 90.00    | 0.47                                       | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R4            | 90.42 | 24.32  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R5            | 95.29 | 29.11  | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R6            | 48.36 | 49.59  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R7            | 63.37 | 33.09  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R8            | 66.28 | 33.09  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R9            | 68.39 | 33.09  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| RESET_BUTTON  | 61.65 | 112.72 | 180.00   | BUTTON_CUSYOMV1                            | MYBUTTON                                  |
| R_BOOT        | 40.69 | 112.21 | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R_CAPACITOR   | 110.43 | 58.27  | 90.00    | 15                                         | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R_CHANGE      | 80.69 | 112.21 | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R_CL1         | 97.01 | 29.09  | 90.00    | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| R_RESET       | 65.75 | 112.21 | 270.00   | 10K                                        | ESP32_WROVER_EAGLE-LTSPICE_R0402          |
| SENSOR2       | 169.62 | 16.39  | 90.00    | ESP32_WROVER_BME680_BME680                 | ESP32_WROVER_BME680_PSON80P300X300X100-8N|
| SJ1           | 85.68 | 15.95  | 0.00     |                                            | SJ                                        |
| TP1           | 22.19 | 74.38  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP10          | 38.29 | 19.76  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP11          | 47.59 | 19.38  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP12          | 101.89 | 101.50 | 0.00     | TPTP20R                                    | TP20R                                     |
| TP13          | 43.19 | 19.52  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP14          | 170.59 | 68.94  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP15          | 133.69 | 14.06  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP16          | 138.59 | 14.08  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP17          | 128.49 | 14.20  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP2           | 25.39 | 74.30  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP3           | 167.29 | 69.02  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP4           | 101.79 | 97.94  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP5           | 101.79 | 110.66 | 0.00     | TPTP20R                                    | TP20R                                     |
| TP6           | 96.39 | 110.68 | 0.00     | TPTP20R                                    | TP20R                                     |
| TP7           | 90.89 | 110.60 | 0.00     | TPTP20R                                    | TP20R                                     |
| TP8           | 124.09 | 14.22  | 0.00     | TPTP20R                                    | TP20R                                     |
| TP9           | 101.89 | 94.34  | 0.00     | TPTP20R                                    | TP20R                                     |
| U1            | 52.62 | 57.34  | 0.00     | W25Q512JVEIQ                               | SON127P600X800X80-9N                      |
| U2            | 28.74 | 57.95  | 90.00    | ESP32-C6-WROOM-1-N8                        | XCVR_ESP32-C6-WROOM-1-N8                  |
| U3            | 100.84 | 52.32  | 270.00   | DS3231SN#                                  | SOIC127P1032X265-16N                      |
| U4            | 143.14 | 61.46  | 0.00     | MAX17048G+T10                              | SON50P200X200X80-9N                       |

## 7. Concluzii
- ESP32-C6 este nucleul sistemului, cu 3 interfețe SPI (Flash, SD, E-Paper) și I2C partajat (BME688 + DS3231)
- BME688 oferă date ambientale, iar DS3231 menține timpul precis
- Consum redus în sleep (~5 µA) permite funcționare pe baterie >1 lună

Probleme : 
 - Nu pot genera fiserul .step (se blocheaza Fusion-ul cand incerc)
