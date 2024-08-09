# CodigoIot_GasCare
Prototipo de un sistema de detector de fugas de gas,  monitoreo de humedad y temperatura en cocinas. 
## Introducción
El proyecto se enfocará en implementar prototipo para un sistema de detección de fugas de gas confiable y preciso mediante el uso de sensores, microcontroladores y codigo adecuado.

Se desarrollará un mecanismo de cierre automático que responda de manera rápida y efectiva ante la detección de una fuga de gas.

## Material necesario
- RasberryPi4
- Esp32CAM
- Sensor de gas LP MQ-6
- Jumpers Hembra -Hembra
- FTD1232 USB
- Sensor de Temperatura Y Humedad DHT11
- Electro Valvúla solenoide 12v
- Relay

## Material de referencia
- [rasberry pi OS](https://www.raspberrypi.com/software/operating-systems/ "rasberry pi OS")
- [Node-Red](https://nodered.org/docs/getting-started/local "Node-Red")
- [Node-red Dashboard](https://flows.nodered.org/node/node-red-dashboard "Node-red Dashboard")
-
## Conexión de RasberryPi y DHT11

| RasberryPI4      | DHT11 |
| --------- | ------|
|  3.3V    |   Vcc  |
| GND     |    GND |
| 114PIN      |     DAT |

## Conexión ESP32CAM y FTD1232 USB

| ESP32CAM      | FTD1232 USB |
| --------| ----|
| 5V  | Vcc |
| GND     |   GND |
| UOT      |    RX |
| UOR      |    TX |
| 1O0-   GND |

## Conexión de ESP2CAM y MQ-6
| ESP32CAM      | MQ-6 |
| --------- | ------|
|  3.3V    |   Vcc  |
| GND     |    GND |
| 114PIN      |     ADC |

## Codigo  de conexión ESP32CAM y M1-6

```

#define VALVE_PIN 7  // Pin digital que controlará el relé de la válvula
#define MQ6_PIN 14

float R0 = 9.83; // Valor de la resistencia del sensor en aire limpio
float threshold = 500; // Umbral de PPM para encender/apagar la válvula

void setup() {
  Serial.begin(115200);
  pinMode(MQ6_PIN, INPUT);
  pinMode(VALVE_PIN, OUTPUT);
  digitalWrite(VALVE_PIN, LOW); // Asegúrate de que la válvula esté apagada al inicio
}

void loop() {
  float sensor_volt;
  float RS_gas; // Valor de la resistencia del sensor en presencia de gas
  float ratio; // Relación RS/R0
  float ppm_log;
  float ppm;

  sensor_volt = analogRead(MQ6_PIN);
  sensor_volt = sensor_volt / 4095 * 3.3; // Conversión a voltaje
  RS_gas = (3.3 - sensor_volt) / sensor_volt; // Conversión a resistencia del sensor
  ratio = RS_gas / R0; // Cálculo de la relación RS/R0
  
  ppm_log = (log10(ratio) - 2.3) / -1.36; // Fórmula de la hoja de datos para LPG
  ppm = pow(10, ppm_log);

  Serial.print("RS/R0 = ");
  Serial.print(ratio);
  Serial.print("\t PPM = ");
  Serial.println(ppm);

  // Control de la válvula basado en el valor de PPM
  if (ppm > threshold) {
    digitalWrite(VALVE_PIN, HIGH); // Encender la válvula (abrir)
  } else {
    digitalWrite(VALVE_PIN, LOW); // Apagar la válvula (cerrar)
  }

  delay(1000);
}

```
