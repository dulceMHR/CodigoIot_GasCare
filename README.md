
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
- [Rasberry pi OS](https://www.raspberrypi.com/software/operating-systems/ "rasberry pi OS")
- [Node-Red](https://nodered.org/docs/getting-started/local "Node-Red")
- [Node-red Dashboard](https://flows.nodered.org/node/node-red-dashboard "Node-red Dashboard")
- [Node Red DHT11](https://flows.nodered.org/node/node-red-contrib-dht-sensor "Node Red DHT11")
- [Configuración del IDE para ESP32CAM](https://edu.codigoiot.com/mod/lesson/view.php?id=4390&pageid=4590&startlastseen=yes "Configuración del IDE para ESP32CAM")
- [Introducción a MQTT (Broker Mosquitto)](https://edu.codigoiot.com/course/view.php?id=851 "Introducción a MQTT (Broker Mosquitto)")


## Conexión de RasberryPi y DHT11

En este proyecto, conectaremos un sensor de temperatura y humedad DHT11 a una Raspberry Pi 4 para leer los datos ambientales. A continuación se describe cómo realizar la conexión:

### Tabla de Conexiones

| Raspberry Pi 4 | DHT11 |
| -------------- | ----- |
| 3.3V           | VCC   |
| GND            | GND   |
| GPIO 17 (PIN 11) | DAT   |

### Descripción de las Conexiones

1. **3.3V a VCC**: Conectamos el pin de 3.3V de la Raspberry Pi al pin de VCC del sensor DHT11. Esto suministra la energía necesaria para que el sensor funcione.

2. **GND a GND**: El pin GND de la Raspberry Pi se conecta al pin GND del DHT11. Esto establece una referencia de tierra común para ambos dispositivos.

3. **GPIO 17 (PIN 11) a DAT**: El pin GPIO 17 (correspondiente al pin físico 11 en la Raspberry Pi) se conecta al pin DAT del sensor DHT11. Este es el pin de datos que permite la comunicación entre la Raspberry Pi y el sensor, enviando la información de temperatura y humedad captada por el DHT11.

###### 

## Conexión entre ESP32-CAM y FTD1232 USB

En este proyecto, conectaremos un módulo ESP32-CAM a un adaptador USB FTD1232 para facilitar la programación y comunicación con el ESP32-CAM desde un ordenador. A continuación se describe cómo realizar la conexión:

### Tabla de Conexiones

| ESP32-CAM  | FTD1232 USB |
| ---------- | ----------- |
| 5V         | VCC         |
| GND        | GND         |
| UOT        | RX          |
| UOR        | TX          |
| IO0 (GPIO 0) a GND |        |

### Descripción de las Conexiones

1. **5V a VCC**: Conectamos el pin de 5V del ESP32-CAM al pin VCC del adaptador FTD1232 USB. Esto suministra la energía necesaria para que el ESP32-CAM funcione.

2. **GND a GND**: El pin GND del ESP32-CAM se conecta al pin GND del FTD1232 USB. Esto establece una referencia de tierra común para ambos dispositivos.

3. **UOT a RX**: El pin UOT del ESP32-CAM se conecta al pin RX del FTD1232 USB. Este es el pin de transmisión de datos desde el ESP32-CAM hacia el adaptador USB.

4. **UOR a TX**: El pin UOR del ESP32-CAM se conecta al pin TX del FTD1232 USB. Este es el pin de recepción de datos desde el adaptador USB hacia el ESP32-CAM.

5. **IO0 (GPIO 0) a GND**: Para poner el ESP32-CAM en modo de programación, es necesario conectar el pin IO0 (GPIO 0) a GND antes de encender el dispositivo. Esto permitirá cargar el código desde el ordenador.

### Diagrama de Conexión

![Diagrama de Conexión](imagenes/conexiones_componentes.jpg)

### Consideraciones Adicionales

- **Modo de Programación**: Asegúrate de que el pin IO0 esté conectado a GND al encender el ESP32-CAM para poder programarlo. Después de cargar el código, puedes desconectar IO0 de GND y reiniciar el ESP32-CAM para que funcione en modo normal.
- **Reseteo**: En algunos casos, puede ser necesario reiniciar el ESP32-CAM manualmente después de cargar el código.



## Conexión de ESP2CAM y MQ-6
| ESP32CAM      | MQ-6 |
| --------- | ------|
|  3.3V    |   Vcc  |
| GND     |    GND |
| 114PIN      |     ADC |

##Conexión de relay con la RasberryPi4

| Relay      |RasberryPi4 |
| --------- | ------|
|  IN    |    GPIO 19  |
| GND     |    GND |
| VCC      |     5V |

## Conexión de relay y la Valvula Solenoide




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
