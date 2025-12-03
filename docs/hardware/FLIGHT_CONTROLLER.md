# Controlador de Vuelo - Flight Controller

## Arquitectura Híbrida

El sistema DronReaper utiliza una configuración híbrida de dos microcontroladores para optimizar el rendimiento:

## STM32F405RG - Controlador de Vuelo Primario

### Especificaciones

| Característica | Valor |
|----------------|-------|
| Núcleo | ARM Cortex-M4F |
| Frecuencia | 168 MHz |
| Flash | 1 MB |
| RAM | 192 KB + 64 KB CCM |
| FPU | Sí (unidad de punto flotante) |
| ADC | 3x 12-bit |
| Timers | 14 |
| UART | 6 |
| SPI | 3 |
| I2C | 3 |

### Conexiones en el STM32F405

```
STM32F405
    │
    ├── SPI1 ──────────► IMU (ICM-20602)
    │
    ├── I2C1 ──────────► Magnetómetro (QMC5883L)
    │                └─► Barómetro (BMP280)
    │                └─► VL53L0X (sensor distancia)
    │
    ├── UART1 ─────────► GPS (uBlox NEO-M8N)
    │
    ├── UART2 ─────────► Receptor RC (ELRS CRSF)
    │
    ├── UART3 ─────────► ESP32-S3 (Telemetría)
    │
    ├── PWM (TIM1-4) ──► ESCs (4-6 canales)
    │
    └── ADC ───────────► Sensor de voltaje/corriente
```

### Firmware Compatible

- **INAV** - Recomendado para ala fija
- **ArduPilot** - Alternativa con más funciones
- **Betaflight** - Solo para multirrotores

---

## ESP32-S3-WROOM - Co-procesador IA/Comunicaciones

### Especificaciones

| Característica | Valor |
|----------------|-------|
| Núcleo | Dual Xtensa LX7 |
| Frecuencia | 240 MHz |
| Flash | 8-16 MB (externo) |
| PSRAM | 2-8 MB (opcional) |
| WiFi | 802.11 b/g/n (2.4 GHz) |
| Bluetooth | BLE 5.0 |
| USB | OTG integrado |
| ADC | 2x 12-bit |
| UART | 3 |
| SPI | 4 |
| I2C | 2 |

### Conexiones en el ESP32-S3

```
ESP32-S3
    │
    ├── UART0 ─────────► STM32F405 (Telemetría)
    │
    ├── UART1 ─────────► A7670E (Módulo LTE)
    │
    ├── WiFi ──────────► Telemetría local / Configuración
    │
    ├── Bluetooth ─────► Configuración / Debug
    │
    ├── SPI (opcional) ► Cámara adicional
    │
    └── I2C ───────────► Sensores secundarios
```

### Funciones del ESP32-S3

1. **Telemetría LTE/WiFi**
   - Envío de datos JSON a servidor
   - Integración con Google Maps API
   - Streaming de datos en tiempo real

2. **Sistema de Redundancia**
   ```
   if (señal_RC_perdida) {
       usar_control_LTE();
   }
   if (señal_LTE_perdida) {
       activar_RTH();  // Return-to-Home por GPS
   }
   ```

3. **IA Básica**
   - Detección de obstáculos
   - Análisis de telemetría
   - Predicciones de vuelo

---

## Pinout Sugerido para PCB

### STM32F405RG (LQFP64)

| Pin | Función | Conexión |
|-----|---------|----------|
| PA5 | SPI1_SCK | IMU CLK |
| PA6 | SPI1_MISO | IMU MISO |
| PA7 | SPI1_MOSI | IMU MOSI |
| PA4 | SPI1_CS | IMU CS |
| PB6 | I2C1_SCL | I2C Bus SCL |
| PB7 | I2C1_SDA | I2C Bus SDA |
| PA9 | USART1_TX | GPS RX |
| PA10 | USART1_RX | GPS TX |
| PA2 | USART2_TX | RC RX |
| PA3 | USART2_RX | RC TX |
| PB10 | USART3_TX | ESP32 RX |
| PB11 | USART3_RX | ESP32 TX |
| PA0-PA3 | TIM2_CH1-4 | PWM ESC 1-4 |
| PB0-PB1 | TIM3_CH3-4 | PWM ESC 5-6 |

### ESP32-S3-WROOM

| GPIO | Función | Conexión |
|------|---------|----------|
| GPIO43 | UART0_TX | STM32 RX |
| GPIO44 | UART0_RX | STM32 TX |
| GPIO17 | UART1_TX | A7670E RX |
| GPIO18 | UART1_RX | A7670E TX |
| GPIO4 | A7670E_PWR | Control encendido LTE |
| GPIO5 | A7670E_RST | Reset LTE |
| GPIO8 | I2C_SDA | Bus I2C secundario |
| GPIO9 | I2C_SCL | Bus I2C secundario |

---

## Comunicación Inter-procesador

### Protocolo STM32 ↔ ESP32

Comunicación UART a 115200 baudios con protocolo JSON simplificado:

**STM32 → ESP32 (Telemetría de vuelo):**
```json
{
  "lat": 19.432608,
  "lon": -99.133209,
  "alt": 150.5,
  "speed": 25.3,
  "heading": 180,
  "battery": 78,
  "gps_fix": 3,
  "rc_rssi": 85
}
```

**ESP32 → STM32 (Comandos LTE):**
```json
{
  "cmd": "goto",
  "lat": 19.435000,
  "lon": -99.135000,
  "alt": 100
}
```

---

*Continúa en: [Sensores de Navegación](./NAVIGATION_SENSORS.md)*
