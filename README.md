## 1. Diagrama Bloc a Sistemului
Arhitectura hardware este centrata pe microcontrolerul nRF52840, gestionand distributia puterii, comunicatia cu senzorii si interfata vizuala.

      +----------+          +-----------+          +----------+
      |  USB 5V  | -------->|  BQ25180  | -------->|  RT6160  |
      +----------+          | (Charger) |          | (Reg)    |
                            +-----------+          +----------+
                                  ^                     |
                                  |                     | (3.3V Rail)
                                  v                     v
      +----------+          +-----------+          +----------+
      |   LiPo   | <------> |  BAT Node |          | nRF52840 |
      | Battery  |          +-----------+          |  (MCU)   |
      +----------+                |                +----------+
                                  v                     |
                            +-----------+               |
                            | MAX17048  | <-------------+ (I2C Bus)
                            | (Fuel G.) |               |
                            +-----------+               |
                                                        |
         +----------------------------------------------+
         |               |               |              |
         v (I2C)         v (I2C)         v (SPI)        v (GPIO)
   +-----------+   +-----------+   +-----------+   +------------+
   |  BMA421   |   | DRV2605L  |   |  E-Paper  |   | 3x Buttons |
   |  (Accel)  |   | (Haptic)  |   |  Display  |   +------------+
   +-----------+   +-----------+   +-----------+
                         |               ^
                         v               | (Power Gate)
                   +-----------+   +-----------+
                   |Vibe Motor |   | SI2301CDS |
                   +-----------+   |  (PFET)   |
                                   +-----------+

## 2. Bill of Materials (BOM)
Componentele sunt selectate pentru catalogul de asamblare JLCPCB.

| Functie      | Componenta     | Capsula    |
|--------------|----------------|------------|
| MCU          | nRF52840-QIAA  | aQFN73     | 
| Charger      | BQ25180YBGR    | DSBGA-8    |
| Regulator    | RT6160AWSC     | WLCSP-15   |
| Fuel Gauge   | MAX17048G+T10  | DFN-8      |
| Accel        | BMA421         | LGA-12     |
| Haptic Drv   | DRV2605LDGSR   | VSSOP-10   |
| Vibe Motor   | LCM1027B3605F  | Wire       |
| EPD PFET     | SI2301CDS      | SOT-23     |

## 3. Functionalitate Hardware in Detaliu

### Managementul Puterii (Power Tree)
[cite_start]Sistemul utilizeaza o arhitectura tip "power-path":
* **BQ25180**: Gestioneaza incarcarea bateriei LiPo de 250 mAh si asigura tranzitia automata intre alimentarea USB si baterie.
* **RT6160**: Regulator buck-boost setat pentru o iesire de 3.3V (VSEL low), asigurand stabilitatea MCU-ului pe toata durata de descarcare a bateriei.
* **MAX17048**: Monitorizeaza precis tensiunea bateriei si starea de incarcare (SOC) prin masurare directa pe VBAT.

### Senzori si Interfete
* **Magistrala I2C**: Partajata intre accelerometru, fuel gauge, charger, regulator si driverul haptic.
* **Interfata SPI**: Utilizata exclusiv pentru controller-ul ecranului e-paper.
* **BMA421**: Detecteaza pașii mereu activ si trimite intreruperi hardware (INT1) catre MCU pentru wake-up.
* **Haptics**: Driverul DRV2605L controleaza motorul ERM pentru alerte tactile, fiind activat (EN) doar in timpul utilizarii pentru a economisi energie.

### Calcule si Eficienta Consumului
* **Power Gating**: Un PFET (SI2301) controlat de MCU (P1.01) intrerupe alimentarea ecranului e-paper (VEPD) cand acesta nu se afla intr-un ciclu de refresh.
* **Tinta Autonomie**: >= 30 de zile cu refresh de ecran o data pe minut si maxim 50 de notificari zilnice.

## 4. Alocarea Pinilor nRF52840

Pinii au fost selectati pentru a optimiza rutarea pe un PCB cu 4 straturi.

| Componenta      | Pin MCU       | Justificare / Functie |
|-----------------|---------------|-----------------------|
| **I2C SDA** | P0.06  | Comunicatie date senzori si IC-uri power. |
| **I2C SCL** | P0.07 | Ceas magistrala I2C (10k pull-up extern). |
| **SPI SCK** | P0.02 | Ceas SPI pentru transfer rapid display. |
| **SPI MOSI** | P0.03 | Date seriale catre ecran e-paper. |
| **SPI CS** | P0.05 | Selectie chip afisaj (Chip Select). |
| **EPD DC** | P0.15 | Selectie Data/Command pentru display. |
| **EPD RST** | P0.16 | Reset hardware display. |
| **EPD Busy** | P0.17 | Monitorizare stare procesare refresh ecran. |
| **EPD Power** | P1.01 | Poarta PFET pentru management alimentare VEPD. |
| **Accel INT** | P0.08 | INT1 (wake primar pentru pedometru). |
| **Accel INT2** | P1.08 | INT2 pentru evenimente secundare. |
| **Fuel Gauge** | P0.10 | ALRT pentru prag baterie descarcata. |
| **Charger INT** | P0.11 | Intrerupere pentru evenimente de incarcare. |
| **Haptic EN** | P0.12 | Enable pentru oprirea driverului haptic. |
| **Buttons** | P0.13, P0.14 | Up si Down (active low, wake source). |
| **Button Ent** | P1.00 | Enter / Esc (active low). |
