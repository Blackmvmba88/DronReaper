# Software y Algoritmos

## Arquitectura de Software

El sistema DronReaper utiliza una arquitectura distribuida entre dos microcontroladores con funciones específicas.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ARQUITECTURA DE SOFTWARE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────┐   ┌─────────────────────────────┐        │
│   │       STM32F405             │   │        ESP32-S3             │        │
│   │   (Control de Vuelo)        │   │   (IA / Comunicaciones)     │        │
│   │                             │   │                             │        │
│   │  ┌─────────────────────┐    │   │  ┌─────────────────────┐    │        │
│   │  │  INAV / ArduPilot   │    │   │  │  Firmware Custom    │    │        │
│   │  └─────────────────────┘    │   │  └─────────────────────┘    │        │
│   │           │                 │   │           │                 │        │
│   │  ┌────────┴────────┐        │   │  ┌────────┴────────┐        │        │
│   │  │                 │        │   │  │                 │        │        │
│   │  ▼                 ▼        │   │  ▼                 ▼        │        │
│   │ PID            Navegación   │   │ Telemetría       IA        │        │
│   │ Control        GPS          │   │ LTE/WiFi         Detección │        │
│   │                             │   │                             │        │
│   │  ┌─────────────────────┐    │   │  ┌─────────────────────┐    │        │
│   │  │     Failsafe        │    │   │  │   Redundancia       │    │        │
│   │  │   (RTH, Land)       │    │   │  │   Comunicaciones    │    │        │
│   │  └─────────────────────┘    │   │  └─────────────────────┘    │        │
│   │                             │   │                             │        │
│   └──────────────┬──────────────┘   └──────────────┬──────────────┘        │
│                  │                                  │                       │
│                  └──────────── UART ───────────────┘                       │
│                         (Protocolo JSON)                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Software STM32F405 - Controlador de Vuelo

### Opciones de Firmware

#### INAV (Recomendado para Ala Fija)

| Característica | Descripción |
|----------------|-------------|
| Tipo | Firmware de vuelo open-source |
| Especialidad | Alas fijas y navegación GPS |
| Licencia | GPL-3.0 |
| Configuración | INAV Configurator |

**Características principales:**
- Control PID avanzado para alas fijas
- Navegación por waypoints
- Return-to-Home (RTH)
- Failsafe configurable
- Soporte para múltiples configuraciones de ala

#### ArduPilot (Alternativa)

| Característica | Descripción |
|----------------|-------------|
| Tipo | Autopilot completo |
| Especialidad | Misiones complejas |
| Licencia | GPL-3.0 |
| Configuración | Mission Planner / QGroundControl |

---

### Control PID

El sistema PID (Proporcional-Integral-Derivativo) controla la estabilidad del vuelo.

```
                    ┌─────────────┐
   Setpoint ──────►│             │
       +           │  Controlador│────────► Salida
       │    ┌─────►│     PID     │         (Servo/Motor)
       │    │      │             │
       ▼    │      └─────────────┘
     ┌───┐  │
     │ - │  │
     └─┬─┘  │
       │    │
       ▼    │
   ┌───────┐│
   │Sensor ├┘
   │(IMU)  │
   └───────┘
```

#### Fórmula PID

```
u(t) = Kp·e(t) + Ki·∫e(t)dt + Kd·de(t)/dt

Donde:
  u(t) = Señal de control
  e(t) = Error (setpoint - valor actual)
  Kp   = Ganancia proporcional
  Ki   = Ganancia integral
  Kd   = Ganancia derivativa
```

#### Parámetros Típicos para Ala Fija

| Eje | Kp | Ki | Kd |
|-----|----|----|------|
| Roll | 0.45 | 0.05 | 0.02 |
| Pitch | 0.50 | 0.06 | 0.025 |
| Yaw | 0.35 | 0.04 | 0.01 |

---

### Navegación Autónoma

#### Waypoints GPS

```
┌─────────────────────────────────────────────────────────────┐
│                    MISIÓN CON WAYPOINTS                     │
│                                                             │
│            WP2 ●─────────────────● WP3                     │
│               ╱                     ╲                       │
│              ╱                       ╲                      │
│     HOME ●──╱                         ╲──● WP4             │
│     (RTH) ╲                           ╱                     │
│            ╲                         ╱                      │
│             ╲──────────────────────╱                       │
│                      WP1                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Estructura de Waypoint

```c
typedef struct {
    int32_t lat;          // Latitud × 10^7
    int32_t lon;          // Longitud × 10^7
    int32_t alt;          // Altitud en cm
    uint16_t speed;       // Velocidad en cm/s
    uint8_t action;       // Acción al llegar
    uint8_t flag;         // Flags especiales
} waypoint_t;

// Acciones disponibles
enum WP_ACTION {
    WP_ACTION_WAYPOINT = 1,
    WP_ACTION_HOLD = 2,
    WP_ACTION_RTH = 3,
    WP_ACTION_LAND = 8
};
```

---

### Sistema Failsafe

#### Jerarquía de Failsafe

```
┌─────────────────────────────────────────────────────────────┐
│                   FAILSAFE HIERARCHY                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. RC Signal Lost                                         │
│      └── Wait 2 seconds                                     │
│          └── If still lost → Activate LTE Control           │
│                                                             │
│   2. LTE Connection Lost                                    │
│      └── Wait 5 seconds                                     │
│          └── If still lost → Activate RTH                   │
│                                                             │
│   3. GPS Signal Lost                                        │
│      └── Hold current heading                               │
│          └── Attempt to regain GPS                          │
│              └── If timeout → Emergency Land                │
│                                                             │
│   4. Low Battery                                            │
│      └── < 25% → Warning + RTH suggestion                   │
│      └── < 15% → Forced RTH                                 │
│      └── < 10% → Emergency Land                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Configuración INAV Failsafe

```
# CLI Commands
set failsafe_delay = 10              # 1 segundo
set failsafe_off_delay = 200         # 20 segundos
set failsafe_procedure = RTH         # Return to Home
set failsafe_recovery_delay = 5      # 0.5 segundos
```

---

## Software ESP32-S3 - Co-procesador

### Estructura del Firmware

```
/esp32_firmware
├── main/
│   ├── main.c                 # Punto de entrada
│   ├── telemetry.c           # Telemetría LTE/WiFi
│   ├── lte_modem.c           # Control A7670E
│   ├── wifi_manager.c        # Gestión WiFi
│   ├── fc_protocol.c         # Comunicación con STM32
│   ├── failsafe.c            # Sistema de redundancia
│   └── ai_detection.c        # IA básica
├── components/
│   ├── cJSON/                # Parser JSON
│   └── http_client/          # Cliente HTTP
└── CMakeLists.txt
```

### Telemetría JSON

#### Datos enviados al servidor

```json
{
    "drone_id": "DR001",
    "timestamp": 1699459200,
    "position": {
        "lat": 19.432608,
        "lon": -99.133209,
        "alt": 150.5,
        "heading": 180
    },
    "flight": {
        "speed": 25.3,
        "vario": 1.2,
        "mode": "NAV_WP"
    },
    "battery": {
        "voltage": 15.2,
        "current": 12.5,
        "percent": 78,
        "mah_used": 650
    },
    "signals": {
        "rc_rssi": 85,
        "lte_rssi": -75,
        "gps_sats": 12,
        "gps_hdop": 1.2
    },
    "status": {
        "armed": true,
        "failsafe": false,
        "error_code": 0
    }
}
```

### Sistema de Redundancia de Comunicaciones

```c
typedef enum {
    COMM_RC_ACTIVE,
    COMM_LTE_ACTIVE,
    COMM_AUTONOMOUS
} comm_state_t;

comm_state_t current_comm_state = COMM_RC_ACTIVE;

void check_communication_status() {
    if (is_rc_signal_valid()) {
        current_comm_state = COMM_RC_ACTIVE;
        // Control normal por RC
    } 
    else if (is_lte_connected()) {
        current_comm_state = COMM_LTE_ACTIVE;
        send_warning("RC_LOST_LTE_ACTIVE");
        // Permitir control por LTE
    } 
    else {
        current_comm_state = COMM_AUTONOMOUS;
        send_warning("ALL_COMMS_LOST_RTH");
        // Enviar comando RTH al FC
        send_fc_command(CMD_RTH);
    }
}
```

---

## IA Básica - Detección de Obstáculos

### Usando Sensor VL53L0X

```c
#include "vl53l0x.h"

#define OBSTACLE_THRESHOLD_MM 500  // 50cm

void check_obstacles() {
    uint16_t distance = vl53l0x_read_distance();
    
    if (distance < OBSTACLE_THRESHOLD_MM && distance > 0) {
        // Obstáculo detectado
        obstacle_detected = true;
        
        // Opciones de respuesta:
        // 1. Alertar al piloto
        send_telemetry_warning("OBSTACLE_DETECTED");
        
        // 2. Reducir velocidad automáticamente
        send_fc_command(CMD_REDUCE_SPEED);
        
        // 3. Si muy cerca, ascender
        if (distance < 200) {
            send_fc_command(CMD_CLIMB);
        }
    }
}
```

---

## Panel de Control (Ground Station)

### Características del Dashboard

- **Mapa en tiempo real** (Google Maps API)
- **Indicadores de vuelo**: Altitud, velocidad, rumbo
- **Estado de batería**: Voltaje, corriente, porcentaje
- **Video FPV** (si hay streaming)
- **Señales**: RC RSSI, LTE RSSI, satélites GPS
- **Alertas y warnings**

### Ejemplo de Interfaz Web

```html
<!DOCTYPE html>
<html>
<head>
    <title>DronReaper Control</title>
    <script src="https://maps.googleapis.com/maps/api/js?key=API_KEY"></script>
</head>
<body>
    <div id="map"></div>
    <div id="telemetry">
        <p>Altitud: <span id="alt">---</span> m</p>
        <p>Velocidad: <span id="speed">---</span> km/h</p>
        <p>Batería: <span id="battery">---</span>%</p>
    </div>
    <script>
        // WebSocket para telemetría en tiempo real
        const ws = new WebSocket('ws://servidor:8080');
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            updateTelemetry(data);
            updateMap(data.position);
        };
    </script>
</body>
</html>
```

---

## Herramientas de Desarrollo

| Herramienta | Uso |
|-------------|-----|
| INAV Configurator | Configuración FC |
| ESP-IDF | Desarrollo ESP32 |
| STM32CubeIDE | Desarrollo STM32 (si custom) |
| VS Code + PlatformIO | IDE alternativo |
| Logic Analyzer | Debug UART/SPI/I2C |

---

*Ver esquemáticos en: [/docs/schematics/](../schematics/)*
