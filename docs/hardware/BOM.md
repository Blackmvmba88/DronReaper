# Bill of Materials (BOM)

## Lista de Componentes para PCB DronReaper

### Microcontroladores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | U1 | MCU ARM Cortex-M4 168MHz | LQFP-64 | STMicroelectronics | STM32F405RGT6 |
| 1 | U2 | MCU WiFi/BLE Dual Core | Module | Espressif | ESP32-S3-WROOM-1 |

### Sensores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | U3 | IMU 6-axis Gyro+Accel | LGA-14 | TDK InvenSense | ICM-20602 |
| 1 | U4 | Magnetómetro 3-axis | LGA-16 | QST | QMC5883L |
| 1 | U5 | Barómetro | LGA-8 | Bosch | BMP280 |
| 1 | U6 | Sensor ToF | Module | STMicroelectronics | VL53L0X |
| 1 | U7 | Sensor de corriente | SOIC-8 | Texas Instruments | INA219AIDCNR |

### Comunicaciones

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | U8 | Módulo LTE Cat-1 | Module | SIMCOM | A7670E |
| 1 | - | GPS Module (externo) | - | u-blox | NEO-M8N |
| 1 | - | Receptor RC (externo) | - | BetaFPV | ELRS Nano RX |

### Reguladores de Voltaje

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | U9 | Buck Regulator 5V 3A | SOIC-8 | MPS | MP1584EN-LF-Z |
| 1 | U10 | LDO 3.3V 1A | SOT-223 | AMS | AMS1117-3.3 |

### Cristales y Osciladores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | Y1 | Cristal 8MHz (STM32) | HC49 | - | 8.000MHz |
| 1 | Y2 | Cristal 32.768kHz (RTC) | 2x1.2mm | - | 32.768kHz |

### Capacitores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 20 | C1-C20 | Cap 100nF 50V | 0402 | - | 100nF X7R |
| 4 | C21-C24 | Cap 10µF 25V | 0805 | - | 10µF X5R |
| 2 | C25-C26 | Cap 22µF 10V | 0805 | - | 22µF X5R |
| 2 | C27-C28 | Cap 100µF 25V | 1206 | - | 100µF Electrolítico |
| 2 | C29-C30 | Cap 22pF (cristal) | 0402 | - | 22pF C0G |

### Resistores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 4 | R1-R4 | Resistor 4.7kΩ (I2C pull-up) | 0402 | - | 4.7kΩ 1% |
| 10 | R5-R14 | Resistor 10kΩ | 0402 | - | 10kΩ 1% |
| 4 | R15-R18 | Resistor 1kΩ (LEDs) | 0402 | - | 1kΩ 5% |
| 2 | R19-R20 | Resistor 100Ω | 0402 | - | 100Ω 1% |

### Inductores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | L1 | Inductor 10µH 3A (Buck) | 5x5mm | - | 10µH |
| 2 | L2-L3 | Ferrite Bead | 0402 | - | 120Ω@100MHz |

### Diodos

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | D1 | Schottky 40V 3A | SMA | - | SS34 |
| 1 | D2 | TVS 20V | SMB | - | SMBJ20A |
| 4 | LED1-LED4 | LED Indicador | 0603 | - | Verde/Rojo/Azul |

### Conectores

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | J1 | Conector Batería XT60 | THT | - | XT60PW-M |
| 2 | J2-J3 | JST-GH 6 pin (GPS, RC) | SMD | JST | GHR-06V-S |
| 3 | J4-J6 | JST-GH 4 pin (I2C, UART) | SMD | JST | GHR-04V-S |
| 1 | J7 | USB-C (programación) | SMD | - | USB-C 16pin |
| 1 | J8 | Conector SWD 4-pin | THT 2.54mm | - | Header 4P |
| 4 | J9-J12 | Conector ESC (3-pin) | JST-XH 3P | - | XH-3P |
| 1 | J13 | Conector IPEX LTE | IPEX | - | IPEX MHF |
| 1 | J14 | Conector IPEX WiFi | IPEX | - | IPEX MHF |

### Transistores y MOSFETs

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 2 | Q1-Q2 | MOSFET N-Ch 30V 5A | SOT-23 | - | SI2302 |

### Fusibles

| Qty | Referencia | Descripción | Footprint | Fabricante | Part Number |
|-----|------------|-------------|-----------|------------|-------------|
| 1 | F1 | PTC Resettable 3A | 1206 | - | MF-MSMF250-2 |

---

## Componentes Externos (No en PCB)

| Qty | Descripción | Notas |
|-----|-------------|-------|
| 1 | GPS Module uBlox NEO-M8N | Con antena cerámica |
| 1 | Receptor ELRS | BetaFPV o similar |
| 1 | Batería LiPo 4S 3000mAh | XT60 |
| 4-6 | ESC 30A | Según motores |
| 4-6 | Motor Brushless | Según airframe |
| 1 | Cámara FPV | Opcional |
| 1 | VTX 5.8GHz | Opcional |
| 1 | Antena LTE | Whip o PCB |
| 1 | Antena GPS | Cerámica activa |
| 1 | Antena WiFi | PCB o IPEX dipolo |

---

## Notas de Ensamblaje

### Orden de Soldadura Recomendado

1. **Primero**: Componentes pasivos pequeños (0402, 0603)
2. **Segundo**: ICs (STM32, sensores)
3. **Tercero**: Módulos (ESP32, LTE)
4. **Cuarto**: Conectores
5. **Último**: Componentes grandes (XT60, inductores)

### Verificación Pre-Energización

- [ ] Verificar continuidad VCC-GND (no debe haber corto)
- [ ] Verificar conexiones I2C (SDA, SCL con pull-ups)
- [ ] Verificar polaridad de diodos y capacitores electrolíticos
- [ ] Verificar orientación de ICs (pin 1)

---

## Proveedores Sugeridos

| Proveedor | Tipo | Notas |
|-----------|------|-------|
| LCSC | Componentes China | Económico, amplio catálogo |
| DigiKey | Componentes USA | Amplio stock, rápido |
| Mouser | Componentes USA | Técnico, datasheets |
| AliExpress | Módulos | Económico, tiempo de envío |
| Banggood | Drones/RC | Partes específicas drones |

---

## Costo Estimado (Solo PCB + Componentes)

| Categoría | Costo Estimado |
|-----------|----------------|
| PCB (5 unidades) | $15-25 USD |
| Microcontroladores | $15-20 USD |
| Sensores | $20-30 USD |
| Módulo LTE | $15-25 USD |
| Reguladores | $5-10 USD |
| Pasivos | $10-15 USD |
| Conectores | $10-15 USD |
| **Total PCB** | **$90-140 USD** |

### Componentes Externos Adicionales

| Componente | Costo Estimado |
|------------|----------------|
| GPS Module | $15-30 USD |
| Receptor ELRS | $15-25 USD |
| Batería 4S | $30-50 USD |
| ESCs (4x) | $40-80 USD |
| Motores (4x) | $40-80 USD |
| FPV System | $50-100 USD |
| **Total Externo** | **$190-365 USD** |

### **TOTAL PROYECTO: $280-500 USD**

---

*Ver archivos de diseño en: [/docs/schematics/](../schematics/)*
