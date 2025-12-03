# Sensores de Navegación

## Visión General

El sistema de navegación del DronReaper utiliza múltiples sensores para proporcionar datos precisos de posición, orientación y altitud.

---

## IMU - Unidad de Medición Inercial

### ICM-20602 (Recomendado)

| Característica | Valor |
|----------------|-------|
| Fabricante | TDK InvenSense |
| Giroscopio | ±2000°/s |
| Acelerómetro | ±16g |
| Interfaz | SPI / I2C |
| Frecuencia máx | 8 kHz |
| Voltaje | 1.71V - 3.6V |
| Ruido | Muy bajo |

#### Conexión SPI

```
ICM-20602          STM32F405
   VDD ─────────── 3.3V
   GND ─────────── GND
   SCK ─────────── PA5 (SPI1_SCK)
  MISO ─────────── PA6 (SPI1_MISO)
  MOSI ─────────── PA7 (SPI1_MOSI)
   CS  ─────────── PA4 (SPI1_CS)
   INT ─────────── PC4 (EXTI)
```

### Alternativas

| Modelo | Características | Notas |
|--------|----------------|-------|
| MPU6000 | ±2000°/s, ±16g | Clásico, bien soportado |
| BMI270 | Bajo consumo | Usado en drones modernos |
| ICM-42688 | Bajo ruido | Alta precisión |

---

## Magnetómetro - Brújula Digital

### QMC5883L (Recomendado)

| Característica | Valor |
|----------------|-------|
| Fabricante | QST Corporation |
| Rango | ±8 Gauss |
| Resolución | 2 mG |
| Interfaz | I2C |
| Dirección I2C | 0x0D |
| Voltaje | 2.16V - 3.6V |
| ODR máximo | 200 Hz |

#### Conexión I2C

```
QMC5883L           STM32F405
   VCC ─────────── 3.3V
   GND ─────────── GND
   SCL ─────────── PB6 (I2C1_SCL)
   SDA ─────────── PB7 (I2C1_SDA)
  DRDY ─────────── (opcional) PC5
```

#### Notas de Instalación

- **IMPORTANTE**: Montar alejado de motores y cables de potencia
- Orientación: Eje X apuntando hacia adelante del drone
- Calibración obligatoria antes de cada vuelo

### Alternativas

| Modelo | Dirección I2C | Notas |
|--------|---------------|-------|
| IST8310 | 0x0E | Alta precisión |
| HMC5883L | 0x1E | Descontinuado |
| LIS3MDL | 0x1E | STMicro |

---

## Barómetro - Altímetro Barométrico

### BMP280 (Recomendado)

| Característica | Valor |
|----------------|-------|
| Fabricante | Bosch |
| Rango presión | 300-1100 hPa |
| Precisión altitud | ±1 metro |
| Resolución | 0.16 Pa |
| Interfaz | I2C / SPI |
| Dirección I2C | 0x76 / 0x77 |
| Voltaje | 1.71V - 3.6V |

#### Conexión I2C

```
BMP280             STM32F405
   VCC ─────────── 3.3V
   GND ─────────── GND
   SCL ─────────── PB6 (I2C1_SCL)
   SDA ─────────── PB7 (I2C1_SDA)
   SDO ─────────── GND (I2C addr 0x76)
```

### Alternativas

| Modelo | Precisión | Notas |
|--------|-----------|-------|
| MS5611 | ±0.5m | Mayor precisión |
| BMP388 | ±0.5m | Nuevo modelo |
| SPL06-001 | ±0.5m | Alternativa económica |

---

## GPS - Sistema de Posicionamiento Global

### uBlox NEO-M8N (Recomendado)

| Característica | Valor |
|----------------|-------|
| Fabricante | u-blox |
| Constelaciones | GPS + GLONASS + Galileo + BeiDou |
| Precisión horizontal | 2.5m CEP |
| Velocidad máxima | 500 m/s |
| Altitud máxima | 50,000 m |
| Interfaz | UART / I2C |
| Baudrate | 9600-921600 |
| Frecuencia | 10 Hz (configurable) |

#### Conexión UART

```
NEO-M8N            STM32F405
   VCC ─────────── 5V (o 3.3V según módulo)
   GND ─────────── GND
    TX ─────────── PA10 (USART1_RX)
    RX ─────────── PA9 (USART1_TX)
```

#### Configuración Recomendada

```
Baudrate: 115200
Update rate: 10 Hz
SBAS: Habilitado
Constelaciones: GPS + GLONASS
```

### Alternativas

| Modelo | Precisión | Notas |
|--------|-----------|-------|
| NEO-M9N | 1.5m | Nueva generación |
| BN-880 | 2.5m | Incluye brújula |
| Beitian BN-220 | 2.5m | Económico |

---

## Sensor de Distancia - Proximidad

### VL53L0X - Time-of-Flight (Recomendado)

| Característica | Valor |
|----------------|-------|
| Fabricante | STMicroelectronics |
| Tecnología | VCSEL ToF |
| Rango | 50mm - 2000mm |
| Precisión | ±3% |
| Interfaz | I2C |
| Dirección I2C | 0x29 |
| Voltaje | 2.6V - 3.5V |
| FOV | 25° |

#### Conexión I2C

```
VL53L0X            STM32F405
   VIN ─────────── 3.3V
   GND ─────────── GND
   SCL ─────────── PB6 (I2C1_SCL)
   SDA ─────────── PB7 (I2C1_SDA)
  XSHUT ────────── PC6 (control encendido)
  GPIO1 ────────── PC7 (interrupción, opcional)
```

#### Aplicaciones

- Detección de obstáculos cercanos
- Asistencia en aterrizaje
- Evitación de colisiones

### Alternativas

| Modelo | Rango | Notas |
|--------|-------|-------|
| VL53L1X | 4m | Mayor alcance |
| HC-SR04 | 4m | Ultrasonido, más lento |
| TFMini-S | 12m | LiDAR, mayor alcance |

---

## Bus I2C Compartido

Todos los sensores I2C comparten el mismo bus:

```
                    3.3V
                     │
                   ┌─┴─┐
                   │4K7│ (Pull-up)
                   └─┬─┘
     STM32          │              Sensores
   ┌───────┐        │         ┌─────────────┐
   │  PB6  ├────────┼────SCL──┤  QMC5883L   │
   │       │        │         │  (0x0D)     │
   │  PB7  ├────────┼────SDA──┤             │
   └───────┘        │         └─────────────┘
                    │         ┌─────────────┐
                    ├────SCL──┤   BMP280    │
                    │         │  (0x76)     │
                    ├────SDA──┤             │
                    │         └─────────────┘
                    │         ┌─────────────┐
                    ├────SCL──┤  VL53L0X    │
                    │         │  (0x29)     │
                    └────SDA──┤             │
                              └─────────────┘
```

### Direcciones I2C del Sistema

| Dispositivo | Dirección Hex | Dirección 7-bit |
|-------------|---------------|-----------------|
| QMC5883L | 0x0D | 13 |
| BMP280 | 0x76 | 118 |
| VL53L0X | 0x29 | 41 |
| INA219 | 0x40 | 64 |

---

*Continúa en: [Sistema de Comunicaciones](./COMMUNICATIONS.md)*
