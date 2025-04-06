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
    ![Architecture Diagram]([https://raw.githubusercontent.com/mihai1400/Proiect_TSC/main/diagrama.png](https://github.com/mihai1400/Proiect_TSC/blob/main/diagrama.png))

## 6. Concluzii
- ESP32-C6 este nucleul sistemului, cu 3 interfețe SPI (Flash, SD, E-Paper) și I2C partajat (BME688 + DS3231)
- BME688 oferă date ambientale, iar DS3231 menține timpul precis
- Consum redus în sleep (~5 µA) permite funcționare pe baterie >1 lună

