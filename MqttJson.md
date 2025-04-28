
Codigo simples para demostrar a leitura das portas analógicas e publicação em formato Json via MQTT

- Utiliza bitfileds para armazenar em 10 bits a leitura de cada porta analogica
- Imprime em formato Json sem utilizar biblioteca externa

```MqqtJson.ino
#include "common.h"

void setup() {
  Serial.begin(9600);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  
  client.loop();
  
  sensor_t sensor = readSensor();
  String jsonData = getJson(sensor);

  Serial.println(jsonData);
  client.publish(topic, jsonData.c_str());

  delay(10000);
}
```

```common.h
#ifndef __COMMON_H__
#define __COMMON_H__

#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "Wokwi-GUEST";  // Substitua pelo seu SSID
const char* password = "";  // Substitua pela senha do Wi-Fi
const char* mqtt_server = "broker.hivemq.com";  // Substitua pelo endereço do broker MQTT
const char* topic = "wokwi-lab/esp32c3/sensores";  // Tópico MQTT para envio dos dados

WiFiClient espClient;
PubSubClient client(espClient);

typedef struct {
  unsigned int a0 : 10;
  unsigned int a1 : 10;
  unsigned int a2 : 10;
  unsigned int a3 : 10;
} sensor_t;

void setup_wifi();
void reconnect();
sensor_t readSensor();
String getJson(sensor_t sensor);

#endif // __COMMON_H__
```


```common.ino
#include "common.h"

void setup_wifi() {
  Serial.print("Conectando ao WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi conectado!");
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Conectando ao MQTT...");
    if (client.connect("ESP32Client")) {
      Serial.println("Conectado!");
    } else {
      Serial.print("Falha. Código: ");
      Serial.print(client.state());
      Serial.println(" Tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}

sensor_t readSensor() {
  sensor_t sensor;
  
  sensor.a0 = analogRead(A0); delay(50);
  sensor.a1 = analogRead(A1); delay(50);
  sensor.a2 = analogRead(A2); delay(50);
  sensor.a3 = analogRead(A3); delay(50);

  return sensor;
}

String getJson(sensor_t sensor) {
  return String("{\"a0\":") + sensor.a0 + 
         ",\"a1\":" + sensor.a1 + 
         ",\"a2\":" + sensor.a2 + 
         ",\"a3\":" + sensor.a3 + "}";
}
```
