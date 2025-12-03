# Sistema de Comunicaciones

## Visión General

El DronReaper implementa un sistema de comunicaciones redundante con múltiples tecnologías para garantizar conectividad en cualquier situación.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SISTEMA DE COMUNICACIONES                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
│   │  RF 2.4GHz  │  │  GSM/LTE    │  │    WiFi     │  │    FPV    │ │
│   │   (ELRS)    │  │  (A7670E)   │  │  (ESP32)    │  │  (5.8GHz) │ │
│   │             │  │             │  │             │  │           │ │
│   │  Control    │  │  Telemetría │  │  Config/    │  │   Video   │ │
│   │  Primario   │  │  Ilimitada  │  │  Local      │  │   FPV     │ │
│   └─────────────┘  └─────────────┘  └─────────────┘  └───────────┘ │
│         ▲                ▲                ▲                ▲       │
│         │                │                │                │       │
│         └────────────────┴────────────────┴────────────────┘       │
│                              DRONE                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Control RF 2.4GHz - ELRS

### ExpressLRS (Recomendado)

| Característica | Valor |
|----------------|-------|
| Protocolo | CRSF |
| Frecuencia | 2.4 GHz |
| Alcance | 30+ km (con antena directiva) |
| Latencia | ~6ms |
| Potencia TX | 10mW - 1000mW |
| Canales | 12+ |

#### Receptor Compatible

**Ejemplo: BetaFPV ELRS Nano RX**

| Característica | Valor |
|----------------|-------|
| Peso | 0.5g |
| Voltaje | 3.3V - 5V |
| Protocolo | CRSF |
| Antena | Cerámica / Dipolo |

#### Conexión CRSF

```
ELRS RX            STM32F405
   VCC ─────────── 5V
   GND ─────────── GND
    TX ─────────── PA3 (USART2_RX)
    RX ─────────── PA2 (USART2_TX)
```

#### Configuración UART

```
Baudrate: 420000 (CRSF)
Protocol: CRSF
```

---

## Telemetría LTE - Módulo Celular

### A7670E (Recomendado para 4G LTE)

| Característica | Valor |
|----------------|-------|
| Fabricante | SIMCOM |
| Tecnología | 4G LTE Cat-1 |
| Bandas | B1/B3/B5/B7/B8/B20/B28 |
| Velocidad DL | 10 Mbps |
| Velocidad UL | 5 Mbps |
| GNSS | GPS + GLONASS (opcional) |
| Interfaz | UART |
| Voltaje | 3.4V - 4.2V |

#### Conexión UART

```
A7670E             ESP32-S3
  VCC ─────────── VBAT (4V)
  GND ─────────── GND
   TX ─────────── GPIO18 (UART1_RX)
   RX ─────────── GPIO17 (UART1_TX)
PWRKEY ────────── GPIO4
  RST ─────────── GPIO5
 NETLIGHT ──────► LED indicador
```

#### Comandos AT Básicos

```
AT                      // Test
AT+CPIN?               // Estado SIM
AT+CSQ                 // Señal
AT+CGATT=1             // Attach GPRS
AT+CGDCONT=1,"IP","internet"  // APN
AT+HTTPINIT            // Iniciar HTTP
AT+HTTPPARA="URL","http://api.tuservidor.com/telemetry"
AT+HTTPACTION=1        // POST
```

### Alternativas

| Modelo | Tecnología | Notas |
|--------|------------|-------|
| SIM800L | 2G GSM | Económico, baja velocidad |
| SIM7000G | LTE Cat-M1 | Bajo consumo |
| BG95-M3 | LTE Cat-M1/NB1 | Quectel, IoT |

---

## WiFi - ESP32 Integrado

### Características WiFi ESP32-S3

| Característica | Valor |
|----------------|-------|
| Estándar | 802.11 b/g/n |
| Frecuencia | 2.4 GHz |
| Seguridad | WPA/WPA2/WPA3 |
| Modos | STA + AP (simultáneo) |
| Alcance | ~100m (campo abierto) |

#### Modos de Operación

**1. Modo AP (Access Point)**
- SSID: `DronReaper_XXXX`
- IP: `192.168.4.1`
- Uso: Configuración local sin internet

**2. Modo STA (Station)**
- Conecta a WiFi existente
- Uso: Telemetría local, updates

**3. Modo Dual**
- AP + STA simultáneo
- Uso: Configuración + telemetría

#### Código Ejemplo (Arduino/ESP-IDF)

```cpp
// Inicialización WiFi en modo AP
WiFi.mode(WIFI_AP);
WiFi.softAP("DronReaper", "password123");
IPAddress IP = WiFi.softAPIP();
Serial.println(IP);  // 192.168.4.1
```

---

## Video FPV

### Opción 1: Analógico 5.8GHz (Simple)

| Componente | Modelo Ejemplo |
|------------|----------------|
| Cámara | Caddx Ratel 2 |
| VTX | Rush Tank Ultimate |
| Frecuencia | 5.8 GHz |
| Potencia | 25-800mW |
| Canales | 48 (Raceband) |

#### Conexión VTX

```
VTX                Drone
  VCC ─────────── 9-12V (desde PDB)
  GND ─────────── GND
  VID ─────────── Cámara Video Out
  AUD ─────────── (opcional) Audio
 UART ─────────── (SmartAudio) STM32 UART
```

### Opción 2: Digital HD (Raspberry Pi)

| Componente | Modelo Ejemplo |
|------------|----------------|
| Computadora | Raspberry Pi Zero 2 W |
| Cámara | Raspberry Pi Camera V2 |
| Resolución | 1080p |
| Transmisión | WiFi / LTE streaming |

#### Streaming con GStreamer

```bash
# En Raspberry Pi
gst-launch-1.0 v4l2src device=/dev/video0 ! \
    video/x-raw,width=1280,height=720,framerate=30/1 ! \
    x264enc tune=zerolatency ! \
    rtph264pay ! \
    udpsink host=192.168.1.100 port=5000
```

### Opción 3: Digital DJI / HDZero

| Sistema | Latencia | Alcance |
|---------|----------|---------|
| DJI O3 | ~28ms | 10+ km |
| HDZero | ~15ms | 3+ km |

---

## Sistema de Redundancia

### Lógica de Failover

```
┌─────────────────────────────────────────────────────────────┐
│                 PRIORIDAD DE CONTROL                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. CONTROL RC (ELRS 2.4GHz)                              │
│      │                                                      │
│      ▼ Si se pierde...                                      │
│                                                             │
│   2. CONTROL LTE (A7670E)                                  │
│      │                                                      │
│      ▼ Si se pierde...                                      │
│                                                             │
│   3. RETURN-TO-HOME (GPS autónomo)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Implementación en ESP32

```cpp
enum ControlSource {
    CTRL_RC,
    CTRL_LTE,
    CTRL_AUTONOMOUS
};

ControlSource currentControl = CTRL_RC;

void checkControlSource() {
    if (isRCActive()) {
        currentControl = CTRL_RC;
    } else if (isLTEActive()) {
        currentControl = CTRL_LTE;
        sendWarning("RC Lost - Using LTE");
    } else {
        currentControl = CTRL_AUTONOMOUS;
        activateRTH();  // Return-to-Home
        sendWarning("LTE Lost - RTH Activated");
    }
}
```

---

## Conectores de Antena

### IPEX / U.FL

| Sistema | Conector | Tipo Antena |
|---------|----------|-------------|
| ELRS RX | IPEX1 | Dipolo / Cerámica |
| A7670E LTE | IPEX1 | LTE Whip |
| ESP32 WiFi | IPEX1 / PCB | Integrada / Externa |
| GPS | IPEX1 | Cerámica activa |
| FPV VTX | SMA | Dipolo / Patch |

---

*Continúa en: [Sistema de Energía](./POWER_SYSTEM.md)*
