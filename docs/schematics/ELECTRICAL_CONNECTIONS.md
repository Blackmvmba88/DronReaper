# Conexiones Eléctricas del Sistema

## Diagrama General de Conexiones

Este documento describe las conexiones eléctricas entre todos los componentes del sistema DronReaper.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                           DIAGRAMA DE CONEXIONES ELÉCTRICAS                             │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                         │
│   ┌───────────────┐                                                                     │
│   │   BATERÍA     │                                                                     │
│   │   LiPo 4S     │                                                                     │
│   │   14.8V       │                                                                     │
│   └───────┬───────┘                                                                     │
│           │                                                                             │
│           ▼                                                                             │
│   ┌───────────────┐        ┌────────────────────────────────────────────────────────┐  │
│   │     PDB       │        │                      PCB PRINCIPAL                      │  │
│   │               │        │                                                         │  │
│   │   ┌───────────┼────────┼─────────────────────────────────────────────────────┐  │  │
│   │   │           │        │                                                      │  │  │
│   │   │  VBAT ────┼────────┼──► INA219 ──► 12V Buck ──► FPV VTX                  │  │  │
│   │   │           │        │       │                                              │  │  │
│   │   │           │        │       ▼                                              │  │  │
│   │   │           │        │   5V Buck (MP1584EN)                                 │  │  │
│   │   │           │        │       │                                              │  │  │
│   │   │           │        │       ├──► GPS Module                                │  │  │
│   │   │           │        │       ├──► Receptor RC (ELRS)                        │  │  │
│   │   │           │        │       ├──► A7670E LTE                                │  │  │
│   │   │           │        │       └──► 3.3V LDO (AMS1117)                        │  │  │
│   │   │           │        │               │                                      │  │  │
│   │   │           │        │               ├──► STM32F405                         │  │  │
│   │   │           │        │               ├──► ESP32-S3                          │  │  │
│   │   │           │        │               ├──► IMU (ICM-20602)                   │  │  │
│   │   │           │        │               ├──► Magnetómetro (QMC5883L)           │  │  │
│   │   │           │        │               ├──► Barómetro (BMP280)                │  │  │
│   │   │           │        │               └──► VL53L0X                           │  │  │
│   │   │           │        │                                                      │  │  │
│   │   └───────────┼────────┼──────────────────────────────────────────────────────┘  │  │
│   │               │        │                                                         │  │
│   │   ┌───────────┼────────┼─────────────────────────────────────────────────────┐  │  │
│   │   │           │        │                ESC CONNECTIONS                       │  │  │
│   │   │  VBAT ────┼────────┤                                                      │  │  │
│   │   │           │        │    ESC1 ◄──PWM──► Motor 1                           │  │  │
│   │   │           │        │    ESC2 ◄──PWM──► Motor 2                           │  │  │
│   │   │           │        │    ESC3 ◄──PWM──► Motor 3                           │  │  │
│   │   │           │        │    ESC4 ◄──PWM──► Motor 4                           │  │  │
│   │   │           │        │                                                      │  │  │
│   │   └───────────┘        └─────────────────────────────────────────────────────┘  │  │
│   └───────────────┘                                                                 │  │
│                                                                                         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Conexiones del STM32F405

### Diagrama de Pines

```
                            STM32F405RGT6
                         ┌─────────────────┐
                         │                 │
              SPI1_SCK ──┤ PA5         PC0 ├── ADC_VBAT
             SPI1_MISO ──┤ PA6         PC1 ├── ADC_CURR
             SPI1_MOSI ──┤ PA7         PC2 ├── (libre)
              SPI1_CS ───┤ PA4         PC3 ├── (libre)
                         │                 │
            USART1_TX ───┤ PA9         PC4 ├── IMU_INT
            USART1_RX ───┤ PA10        PC5 ├── MAG_DRDY
                         │                 │
            USART2_TX ───┤ PA2         PC6 ├── VL53_XSHUT
            USART2_RX ───┤ PA3         PC7 ├── VL53_INT
                         │                 │
            USART3_TX ───┤ PB10       PC13 ├── LED_STATUS
            USART3_RX ───┤ PB11       PC14 ├── (32kHz Crystal)
                         │            PC15 ├── (32kHz Crystal)
             I2C1_SCL ───┤ PB6             │
             I2C1_SDA ───┤ PB7         PA8 ├── TIM1_CH1 (PWM1)
                         │            PA11 ├── USB_DM
                PWM2 ────┤ PA0        PA12 ├── USB_DP
                PWM3 ────┤ PA1             │
                PWM4 ────┤ PB0        PA13 ├── SWDIO
                PWM5 ────┤ PB1        PA14 ├── SWCLK
                PWM6 ────┤ PB4             │
                         │                 │
                  GND ───┤ VSS        VDD ├─── 3.3V
                         │                 │
                         └─────────────────┘
```

### Tabla de Conexiones STM32

| Pin STM32 | Función | Conexión | Notas |
|-----------|---------|----------|-------|
| PA5 | SPI1_SCK | ICM-20602 SCK | IMU |
| PA6 | SPI1_MISO | ICM-20602 SDO | IMU |
| PA7 | SPI1_MOSI | ICM-20602 SDI | IMU |
| PA4 | SPI1_CS | ICM-20602 CS | IMU |
| PA9 | USART1_TX | GPS RX | 115200 baud |
| PA10 | USART1_RX | GPS TX | 115200 baud |
| PA2 | USART2_TX | ELRS RX | 420000 baud (CRSF) |
| PA3 | USART2_RX | ELRS TX | 420000 baud (CRSF) |
| PB10 | USART3_TX | ESP32 RX | 115200 baud |
| PB11 | USART3_RX | ESP32 TX | 115200 baud |
| PB6 | I2C1_SCL | Bus I2C | 400kHz |
| PB7 | I2C1_SDA | Bus I2C | 400kHz |
| PA0-PA1, PB0-PB1, PB4, PA8 | PWM | ESCs | 400Hz-2kHz |
| PC0 | ADC1_CH10 | Divisor VBAT | 1:10 |
| PC1 | ADC1_CH11 | Sensor corriente | 0-3.3V |
| PC4 | EXTI4 | IMU INT | Interrupción |
| PC13 | GPIO | LED Status | Indicador |

---

## Conexiones del ESP32-S3

### Diagrama de Pines

```
                            ESP32-S3-WROOM
                         ┌─────────────────┐
                         │                 │
                  3V3 ───┤ 3V3         GND ├─── GND
                  EN  ───┤ EN          IO0 ├─── BOOT (pullup)
                         │                 │
            STM32_TX ────┤ IO44       IO43 ├─── STM32_RX
                         │                 │
             LTE_RX  ────┤ IO17       IO18 ├─── LTE_TX
            LTE_PWR  ────┤ IO4        IO5  ├─── LTE_RST
                         │                 │
             I2C_SDA ────┤ IO8        IO9  ├─── I2C_SCL
                         │                 │
            LED_WIFI ────┤ IO2        IO1  ├─── LED_LTE
                         │                 │
                USB_D- ──┤ IO19      IO20  ├─── USB_D+
                         │                 │
                         └─────────────────┘
```

### Tabla de Conexiones ESP32

| Pin ESP32 | Función | Conexión | Notas |
|-----------|---------|----------|-------|
| IO43 | UART0_TX | STM32 RX (PB11) | Telemetría |
| IO44 | UART0_RX | STM32 TX (PB10) | Telemetría |
| IO17 | UART1_TX | A7670E RX | LTE |
| IO18 | UART1_RX | A7670E TX | LTE |
| IO4 | GPIO | A7670E PWRKEY | Control encendido |
| IO5 | GPIO | A7670E RST | Reset LTE |
| IO8 | I2C_SDA | Bus I2C secundario | Opcional |
| IO9 | I2C_SCL | Bus I2C secundario | Opcional |
| IO2 | GPIO | LED WiFi | Indicador |
| IO1 | GPIO | LED LTE | Indicador |
| IO19, IO20 | USB | USB-C | Programación |

---

## Bus I2C Compartido

### Conexión del Bus I2C

```
                                  3.3V
                                   │
                            ┌──────┴──────┐
                            │  4.7kΩ x2   │
                            │  (Pull-ups) │
                            └──────┬──────┘
                                   │
     STM32F405                     │
   ┌───────────┐                   │
   │    PB6    ├───────────────────┼────────────────────────────────── SCL
   │           │                   │
   │    PB7    ├───────────────────┼────────────────────────────────── SDA
   └───────────┘                   │
                                   │
           ┌───────────────────────┴───────────────────────────┐
           │                       │                           │
           ▼                       ▼                           ▼
   ┌───────────────┐      ┌───────────────┐           ┌───────────────┐
   │   QMC5883L    │      │    BMP280     │           │   VL53L0X     │
   │   Addr: 0x0D  │      │   Addr: 0x76  │           │   Addr: 0x29  │
   │               │      │               │           │               │
   │  VCC GND SCL  │      │  VCC GND SCL  │           │  VIN GND SCL  │
   │   │   │   │   │      │   │   │   │   │           │   │   │   │   │
   └───┼───┼───┼───┘      └───┼───┼───┼───┘           └───┼───┼───┼───┘
       │   │   │              │   │   │                   │   │   │
      3.3V GND SCL           3.3V GND SCL                3.3V GND SCL
             SDA                   SDA                         SDA
```

### Tabla de Dispositivos I2C

| Dispositivo | Dirección | Velocidad | VCC |
|-------------|-----------|-----------|-----|
| QMC5883L | 0x0D | 400 kHz | 3.3V |
| BMP280 | 0x76 | 400 kHz | 3.3V |
| VL53L0X | 0x29 | 400 kHz | 3.3V |
| INA219 | 0x40 | 400 kHz | 3.3V |

---

## Conexión SPI - IMU

### ICM-20602 a STM32

```
   ICM-20602                   STM32F405
  ┌──────────┐               ┌───────────┐
  │          │               │           │
  │   VDD    ├───────────────┤   3.3V    │
  │   GND    ├───────────────┤   GND     │
  │          │               │           │
  │   SCK    ├───────────────┤   PA5     │ (SPI1_SCK)
  │   SDI    ├───────────────┤   PA7     │ (SPI1_MOSI)
  │   SDO    ├───────────────┤   PA6     │ (SPI1_MISO)
  │   CS     ├───────────────┤   PA4     │ (SPI1_CS)
  │   INT    ├───────────────┤   PC4     │ (EXTI)
  │          │               │           │
  └──────────┘               └───────────┘
```

**Configuración SPI:**
- Modo: SPI Mode 3 (CPOL=1, CPHA=1)
- Velocidad: 1 MHz (inicialización), 10 MHz (operación)
- Data: MSB First, 8 bits

---

## Conexión UART - GPS

### uBlox NEO-M8N a STM32

```
   NEO-M8N                     STM32F405
  ┌──────────┐               ┌───────────┐
  │          │               │           │
  │   VCC    ├───────────────┤   5V      │ (o 3.3V según módulo)
  │   GND    ├───────────────┤   GND     │
  │          │               │           │
  │   TX     ├───────────────┤   PA10    │ (USART1_RX)
  │   RX     ├───────────────┤   PA9     │ (USART1_TX)
  │          │               │           │
  │   PPS    ├───────────────┤   (opc)   │ (Time Pulse)
  │          │               │           │
  └──────────┘               └───────────┘
```

**Configuración UART:**
- Baudrate: 115200
- Data: 8N1
- Protocolo: NMEA / UBX

---

## Conexión UART - Receptor RC

### ELRS Receiver a STM32

```
   ELRS RX                     STM32F405
  ┌──────────┐               ┌───────────┐
  │          │               │           │
  │   VCC    ├───────────────┤   5V      │
  │   GND    ├───────────────┤   GND     │
  │          │               │           │
  │   TX     ├───────────────┤   PA3     │ (USART2_RX)
  │   RX     ├───────────────┤   PA2     │ (USART2_TX)
  │          │               │           │
  └──────────┘               └───────────┘
```

**Configuración UART:**
- Baudrate: 420000 (CRSF)
- Protocolo: CRSF (Crossfire)

---

## Conexión UART - Módulo LTE

### A7670E a ESP32-S3

```
   A7670E                      ESP32-S3
  ┌──────────┐               ┌───────────┐
  │          │               │           │
  │  VBAT    ├───────────────┤ VBAT (4V) │ (desde batería con regulador)
  │   GND    ├───────────────┤   GND     │
  │          │               │           │
  │    TX    ├───────────────┤   IO18    │ (UART1_RX)
  │    RX    ├───────────────┤   IO17    │ (UART1_TX)
  │          │               │           │
  │  PWRKEY  ├───────────────┤   IO4     │ (Control encendido)
  │   RST    ├───────────────┤   IO5     │ (Reset)
  │          │               │           │
  │ NETLIGHT ├───────────────┤  (LED)    │ (Indicador red)
  │          │               │           │
  │   ANT    ├───────────────┤ [IPEX]    │ (Antena LTE)
  │          │               │           │
  └──────────┘               └───────────┘
```

**Configuración UART:**
- Baudrate: 115200
- Protocolo: Comandos AT

---

## Conexión de Potencia

### Reguladores

```
    BATERÍA (14.8V 4S)
         │
         ▼
    ┌─────────┐
    │  FUSE   │ (10A)
    │  PTC    │
    └────┬────┘
         │
         ├────────────────────────────────────────────► ESCs (VBAT directo)
         │
         ▼
    ┌─────────┐
    │ INA219  │ ─────► I2C (Telemetría V/I)
    └────┬────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│ 12V   │ │  5V   │
│ Buck  │ │ Buck  │
│       │ │MP1584 │
└───┬───┘ └───┬───┘
    │         │
    ▼         ├──► GPS (5V)
  FPV VTX     ├──► Receptor RC (5V)
  (12V)       ├──► A7670E LTE (5V → interno 4V)
              │
              ▼
         ┌───────┐
         │ 3.3V  │
         │ LDO   │
         │AMS1117│
         └───┬───┘
             │
             ├──► STM32F405
             ├──► ESP32-S3
             ├──► ICM-20602
             ├──► QMC5883L
             ├──► BMP280
             └──► VL53L0X
```

---

## Conector JST-GH Pinouts

### Conector GPS (6 pines)

```
┌───┬───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │
└───┴───┴───┴───┴───┴───┘
  │   │   │   │   │   │
  │   │   │   │   │   └── GND
  │   │   │   │   └────── 5V
  │   │   │   └────────── RX (GPS TX)
  │   │   └────────────── TX (GPS RX)
  │   └────────────────── SCL (I2C)
  └────────────────────── SDA (I2C)
```

### Conector I2C (4 pines)

```
┌───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │
└───┴───┴───┴───┘
  │   │   │   │
  │   │   │   └── GND
  │   │   └────── 3.3V
  │   └────────── SCL
  └────────────── SDA
```

### Conector ESC (3 pines)

```
┌───┬───┬───┐
│ 1 │ 2 │ 3 │
└───┴───┴───┘
  │   │   │
  │   │   └── GND
  │   └────── (NC)
  └────────── Signal (PWM)
```

---

*Ver esquemático completo en: [/docs/schematics/](../schematics/)*
