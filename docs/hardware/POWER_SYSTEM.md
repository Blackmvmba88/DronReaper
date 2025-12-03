# Sistema de Energía

## Visión General

El sistema de energía del DronReaper proporciona alimentación regulada a todos los subsistemas con monitoreo de corriente y voltaje en tiempo real.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SISTEMA DE ENERGÍA                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────┐                                                          │
│   │   BATERÍA    │                                                          │
│   │  LiPo 3S-4S  │                                                          │
│   │  11.1-14.8V  │                                                          │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│   │     PDB      │────►│  INA219      │────►│  Telemetría  │               │
│   │ (Distribución│     │  (Sensor V/I)│     │  (Estado bat)│               │
│   └──────┬───────┘     └──────────────┘     └──────────────┘               │
│          │                                                                  │
│    ┌─────┼─────┬─────────────┬─────────────┐                               │
│    │     │     │             │             │                               │
│    ▼     ▼     ▼             ▼             ▼                               │
│  ┌───┐ ┌───┐ ┌─────┐   ┌─────────┐   ┌─────────┐                          │
│  │ESC│ │ESC│ │12V  │   │  5V 3A  │   │3.3V LDO │                          │
│  │ 1 │ │ 2 │ │Reg  │   │  Buck   │   │AMS1117  │                          │
│  └───┘ └───┘ └──┬──┘   └────┬────┘   └────┬────┘                          │
│                 │           │             │                                │
│                 ▼           ▼             ▼                                │
│              ┌─────┐   ┌─────────┐   ┌─────────────┐                      │
│              │ FPV │   │Servos   │   │ STM32       │                      │
│              │ VTX │   │Receptor │   │ ESP32       │                      │
│              └─────┘   │GPS, LTE │   │ Sensores    │                      │
│                        └─────────┘   └─────────────┘                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Batería LiPo

### Especificaciones Recomendadas

| Configuración | Voltaje Nominal | Rango | Aplicación |
|---------------|-----------------|-------|------------|
| 3S (3 celdas) | 11.1V | 9.0V - 12.6V | Drones ligeros |
| 4S (4 celdas) | 14.8V | 12.0V - 16.8V | Mayor potencia |

### Parámetros de Selección

| Parámetro | Recomendación |
|-----------|---------------|
| Capacidad | 2200-5000 mAh |
| Descarga (C) | 25C-45C |
| Peso | Según autonomía deseada |
| Conector | XT60 |

### Niveles de Voltaje

```
4S LiPo (14.8V nominal):

   16.8V ──── 100% Carga completa
    │
   15.2V ──── 75%
    │
   14.8V ──── 50% (Nominal)
    │
   14.0V ──── 25% ⚠️ Alerta
    │
   13.2V ──── 10% ⚠️ Crítico - RTH
    │
   12.0V ──── 0%  🔴 Aterrizaje forzado
```

---

## Power Distribution Board (PDB)

### Funciones

- Distribución de potencia de batería
- Puntos de soldadura para ESCs
- Reguladores integrados (opcional)
- Pads para sensores de corriente

### Conexiones Típicas

```
         BATERÍA (XT60)
              │
              ▼
    ┌─────────────────────┐
    │         PDB         │
    │                     │
    │  [ESC1] [ESC2]     │
    │                     │
    │  [ESC3] [ESC4]     │
    │                     │
    │  [12V]  [5V]  [BAT]│
    └─────────────────────┘
         │     │      │
         │     │      └── Sensor V/I
         │     └── Regulador 5V
         └── Regulador 12V
```

---

## Reguladores de Voltaje

### Regulador 5V - MP1584EN (Buck)

| Característica | Valor |
|----------------|-------|
| Entrada | 4.5V - 28V |
| Salida | 5V (ajustable) |
| Corriente máx | 3A |
| Eficiencia | ~90% |
| Tipo | Buck switching |

#### Esquemático Básico

```
         VIN (12-16V)
            │
            ▼
      ┌─────────────┐
      │  MP1584EN   │
      │             │
      │ VIN    SW  ─┼──┐
      │             │  │
      │ GND    FB  ─┼──┼──► VOUT (5V)
      │             │  │
      │      COMP  ─┼──┘
      └─────────────┘
            │
           GND
```

#### Cargas en 5V

| Dispositivo | Consumo Típico |
|-------------|----------------|
| GPS uBlox | 50 mA |
| Receptor RC | 100 mA |
| Servos | 200-500 mA |
| A7670E LTE | 500 mA pico |
| **Total máx** | **~1.5A** |

---

### Regulador 3.3V - AMS1117-3.3 (LDO)

| Característica | Valor |
|----------------|-------|
| Entrada | 4.5V - 12V |
| Salida | 3.3V fijo |
| Corriente máx | 1A |
| Dropout | 1.2V |
| Tipo | LDO lineal |

#### Esquemático Básico

```
         VIN (5V)
            │
           ┌┴┐
           │ │ 10µF
           └┬┘
            │
      ┌─────┴─────┐
      │ AMS1117   │
      │           │
      │ VIN  VOUT ├───┬──► VOUT (3.3V)
      │           │   │
      │    GND    │  ┌┴┐
      └─────┬─────┘  │ │ 22µF
            │        └┬┘
           GND       GND
```

#### Cargas en 3.3V

| Dispositivo | Consumo Típico |
|-------------|----------------|
| STM32F405 | 100 mA |
| ESP32-S3 | 240 mA (WiFi activo) |
| IMU ICM-20602 | 5 mA |
| BMP280 | 1 mA |
| QMC5883L | 1 mA |
| VL53L0X | 20 mA |
| **Total máx** | **~400 mA** |

---

## Sensor de Corriente - INA219

### Especificaciones

| Característica | Valor |
|----------------|-------|
| Rango voltaje bus | 0-26V |
| Resolución V | 4 mV |
| Rango corriente | ±3.2A (shunt 0.1Ω) |
| Resolución I | 0.1 mA |
| Interfaz | I2C |
| Dirección I2C | 0x40 (configurable) |

### Conexión

```
INA219             STM32F405
  VCC ─────────── 3.3V
  GND ─────────── GND
  SCL ─────────── PB6 (I2C1_SCL)
  SDA ─────────── PB7 (I2C1_SDA)
  VIN+ ────────── Batería (+)
  VIN- ────────── PDB (+) / ESC (+)
```

### Código Ejemplo (Arduino)

```cpp
#include <Wire.h>
#include <Adafruit_INA219.h>

Adafruit_INA219 ina219;

void setup() {
    ina219.begin();
}

void loop() {
    float voltage = ina219.getBusVoltage_V();
    float current = ina219.getCurrent_mA();
    float power = ina219.getPower_mW();
    
    // Calcular porcentaje batería (4S)
    float percent = map(voltage, 12.0, 16.8, 0, 100);
    percent = constrain(percent, 0, 100);
}
```

---

## Regulador 12V (Opcional)

### Para Video FPV

Si usas VTX que requiere 12V:

| Característica | Valor |
|----------------|-------|
| Entrada | 14-20V (4S-5S) |
| Salida | 12V |
| Corriente | 1A |
| Tipo | Buck |

---

## Protecciones

### Protecciones Recomendadas

| Protección | Implementación |
|------------|----------------|
| Polaridad inversa | Diodo Schottky / MOSFET |
| Sobrecorriente | Fusible / PTC |
| Sobrevoltaje | TVS Diode |
| Filtro EMI | Capacitor 100nF en cada IC |

### Esquema de Protección

```
  BATERÍA +
      │
      ▼
   ┌──┴──┐
   │FUSIBLE│ (10A para 4S)
   └──┬──┘
      │
      ▼
   ┌──┴──┐
   │ TVS │ (SMBJ20A para 4S)
   └──┬──┘
      │
      ▼
    A PDB
```

---

## Consumo Total Estimado

### Modo Crucero

| Sistema | Consumo |
|---------|---------|
| Motores (4x) | 8-15A total |
| Electrónica 5V | 1.5A |
| Electrónica 3.3V | 0.4A |
| Video FPV | 0.5A @12V |
| **Total** | **~12-18A** |

### Autonomía Estimada

Con batería 4S 3000mAh:

```
Autonomía = Capacidad / Consumo
Autonomía = 3000mAh / 15A = 12 min (conservador)
Autonomía = 3000mAh / 10A = 18 min (crucero eficiente)
```

---

*Continúa en: [Bill of Materials (BOM)](./BOM.md)*
