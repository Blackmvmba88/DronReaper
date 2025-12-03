# DronReaper

Drone tipo avión de reconocimiento de largo alcance con navegación autónoma, telemetría redundante y capacidades de IA básica.

## 🚀 Características Principales

- **Navegación Autónoma** - Vuelo por waypoints GPS
- **Telemetría Redundante** - RF 2.4GHz + LTE + WiFi
- **Failsafe Inteligente** - Return-to-Home automático
- **IA Básica** - Detección de obstáculos
- **Video FPV** - Analógico o digital HD

## 📋 Arquitectura del Sistema

El sistema DronReaper está compuesto por 6 subsistemas principales:

| Subsistema | Descripción |
|------------|-------------|
| **A. Controlador de Vuelo** | STM32F405 + ESP32-S3 (configuración híbrida) |
| **B. Sensores de Navegación** | IMU, Magnetómetro, Barómetro, GPS, Sensor de distancia |
| **C. Comunicaciones** | RF 2.4GHz (ELRS), LTE (A7670E), WiFi, FPV |
| **D. Sistema de Energía** | LiPo 3S-4S, reguladores, monitoreo de batería |
| **E. Software/Algoritmos** | INAV/ArduPilot, IA en ESP32, Failsafe |
| **F. Conexiones Eléctricas** | PCB integrada con todos los componentes |

## 📁 Documentación

### Arquitectura
- [Arquitectura del Sistema](docs/architecture/SYSTEM_ARCHITECTURE.md)

### Hardware
- [Controlador de Vuelo (Flight Controller)](docs/hardware/FLIGHT_CONTROLLER.md)
- [Sensores de Navegación](docs/hardware/NAVIGATION_SENSORS.md)
- [Sistema de Comunicaciones](docs/hardware/COMMUNICATIONS.md)
- [Sistema de Energía](docs/hardware/POWER_SYSTEM.md)
- [Bill of Materials (BOM)](docs/hardware/BOM.md)

### Software
- [Arquitectura de Software](docs/software/SOFTWARE_ARCHITECTURE.md)

### Esquemáticos
- [Conexiones Eléctricas](docs/schematics/ELECTRICAL_CONNECTIONS.md)

## 🔧 Componentes Principales

### Microcontroladores
- **STM32F405RGT6** - Controlador de vuelo principal
- **ESP32-S3-WROOM** - Co-procesador IA/Comunicaciones

### Sensores
- **ICM-20602** - IMU (Giroscopio + Acelerómetro)
- **QMC5883L** - Magnetómetro
- **BMP280** - Barómetro
- **uBlox NEO-M8N** - GPS + GLONASS + Galileo
- **VL53L0X** - Sensor de distancia ToF

### Comunicaciones
- **ELRS 2.4GHz** - Control RC (30+ km alcance)
- **A7670E** - Módulo LTE Cat-1
- **ESP32 WiFi** - Telemetría local
- **5.8GHz VTX** - Video FPV

## 📊 Diagrama de Bloques

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   BATERÍA   │────►│     PDB     │────►│ REGULADORES │
│  LiPo 3S-4S │     │             │     │  5V / 3.3V  │
└─────────────┘     └──────┬──────┘     └──────┬──────┘
                           │                    │
                           ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │    ESCs     │     │   STM32F405 │◄── IMU, Baro, Mag
                    │  (Motores)  │     │     (FC)    │◄── GPS, RC
                    └─────────────┘     └──────┬──────┘
                                               │
                                               ▼
                                        ┌─────────────┐
                                        │   ESP32-S3  │◄── LTE, WiFi
                                        │   (IA/Com)  │
                                        └─────────────┘
```

## 💰 Costo Estimado

| Categoría | Costo |
|-----------|-------|
| PCB + Componentes | $90-140 USD |
| Componentes Externos | $190-365 USD |
| **Total Proyecto** | **$280-500 USD** |

## 🔗 Redundancia de Comunicaciones

```
1. RC (ELRS 2.4GHz) → Activo
   │
   └─► Si falla...
       │
       2. LTE (A7670E) → Backup
          │
          └─► Si falla...
              │
              3. GPS Autónomo → RTH (Return-to-Home)
```

## 📝 Licencia

Este proyecto es de código abierto. Ver [LICENSE](LICENSE) para más detalles.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un issue o pull request.

---

**DronReaper** - Drone de Reconocimiento de Largo Alcance
