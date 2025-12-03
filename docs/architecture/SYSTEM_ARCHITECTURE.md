# DronReaper - Arquitectura del Sistema

## Descripción General

DronReaper es un drone tipo avión de reconocimiento con 6 subsistemas principales diseñado para operaciones de largo alcance con capacidades de navegación autónoma, telemetría redundante e inteligencia artificial básica.

## Diagrama de Arquitectura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SISTEMA DRONREAPER                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐   │
│   │  A. CONTROLADOR  │     │  B. SENSORES DE  │     │ C. COMUNICACIONES│   │
│   │    DE VUELO      │◄────┤   NAVEGACIÓN     │     │                  │   │
│   │  (STM32F405 +    │     │                  │     │  RF 2.4GHz       │   │
│   │   ESP32-S3)      │     │  IMU, GPS, Baro  │     │  GSM/LTE         │   │
│   └────────┬─────────┘     │  Magnetómetro    │     │  WiFi, FPV       │   │
│            │               └──────────────────┘     └────────┬─────────┘   │
│            │                                                  │             │
│            ▼                                                  │             │
│   ┌──────────────────┐     ┌──────────────────┐              │             │
│   │  D. SISTEMA DE   │     │  E. ALGORITMOS   │◄─────────────┘             │
│   │    ENERGÍA       │────►│   Y SOFTWARE     │                            │
│   │                  │     │                  │                            │
│   │  LiPo 3S-4S      │     │  INAV/ArduPilot  │                            │
│   │  Reguladores     │     │  IA ESP32        │                            │
│   └──────────────────┘     └──────────────────┘                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Subsistemas

### A. Unidad Principal de Control (Flight Controller)

**Configuración Híbrida Recomendada:**

| Componente | Función |
|------------|---------|
| **STM32F405RG** | Controlador de vuelo primario (FC) - control de vuelo, sensores, estabilidad |
| **ESP32-S3-WROOM** | Co-procesador - IA, conexión LTE/GSM, telemetría, Google Maps API |

**Alternativas:**
- STM32F411 / F7 (compatible con ArduPilot/INAV)
- Raspberry Pi Zero 2 W o CM4 (para IA avanzada + procesamiento de video)

### B. Sensores de Navegación

| Sensor | Modelo | Función |
|--------|--------|---------|
| IMU | MPU6000 / ICM-20602 / BMI270 | Giroscopio + Acelerómetro |
| Magnetómetro | QMC5883L / IST8310 | Brújula digital |
| Barómetro | BMP280 / MS5611 | Altímetro barométrico |
| GPS | uBlox NEO-M8N | GPS + GLONASS + Galileo |
| Sensor de distancia | VL53L0X / HC-SR04 | Time-of-Flight / Ultrasonido |

### C. Comunicaciones

Sistema con redundancia múltiple:

| Sistema | Tecnología | Propósito |
|---------|------------|-----------|
| Control RF | ELRS 2.4GHz | Control primario del drone |
| Telemetría ilimitada | GSM/LTE (A7670E/SIM800L/BG95) | Comunicación celular |
| WiFi | ESP32 integrado | Telemetría local + configuración |
| FPV | Analógico 5.8GHz / Digital | Video en tiempo real |

### D. Sistemas de Energía

| Componente | Especificación |
|------------|----------------|
| Batería | LiPo 3S–4S |
| Regulador 5V | 3A para electrónica |
| Regulador 12V | Para cámaras especiales |
| PDB | Power Distribution Board |
| Sensor de corriente | INA219 |

### E. Algoritmos / Software

**En el STM32 (Controlador de Vuelo):**
- Control PID (INAV o ArduPilot)
- Navegación autónoma por waypoints (GPS)
- Failsafe por pérdida de señal (Return-to-Home)
- Motor mixing para ala fija

**En el ESP32 (Co-procesador):**
- Telemetría JSON por LTE
- Google Maps API (geoposición)
- Sistema de redundancia de comunicaciones
- IA: Detección de obstáculos

**Sistema de Redundancia:**
```
Si falta señal RC → usa LTE
Si falta LTE → regresa por GPS (Return-to-Home)
```

## Panel de Control (Ground Station)

Características del panel de misión UAV:
- Mapa Google Maps
- Altura, velocidad, rumbo
- Estado de baterías
- Video FPV
- Estado de señales: RC, WiFi, LTE
- Alertas y predicciones por IA

---

*Ver documentos adicionales para detalles específicos de cada subsistema.*
