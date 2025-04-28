
Codigo simples para demostrar a leitura das portas analógicas e publicação em formato CSV via MQTT

- Utiliza bitfileds para armazenar em 10 bits a leitura de cada porta analogica

```
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "SEU_SSID";
const char* password = "SUA_SENHA";
const char* mqtt_server = "SEU_BROKER_MQTT";
const char* mqtt_topic = "sensores/dados";

WiFiClient espClient;
PubSubClient client(espClient);

struct Sensor { // 4x 10 bit bitfield
  unsigned int a0 : 10;
  unsigned int a1 : 10;
  unsigned int a2 : 10;
  unsigned int a3 : 10;
} sensor;

void connectWiFi() {
  Serial.print("Conectando ao WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("Conectado ao WiFi!");
}

void connectMQTT() {
  Serial.print("Conectando ao MQTT...");
  client.setServer(mqtt_server, 1883);
  while (!client.connected()) {
    if (client.connect("ESP32Client")) {
      Serial.println("Conectado ao MQTT!");
    } else {
      Serial.print("Falha, rc=");
      Serial.print(client.state());
      Serial.println(" Tentando novamente em 5 segundos...");
      delay(5000);
    }
  }
}

void readSensor() {
  sensor.a0 = analogRead(A0); delay(50);
  sensor.a1 = analogRead(A1); delay(50);
  sensor.a2 = analogRead(A2); delay(50);
  sensor.a3 = analogRead(A3); delay(50);
}

void sendCSV() {
  String csvData = String(sensor.a0) + "," + 
                   String(sensor.a1) + "," + 
                   String(sensor.a2) + "," + 
                   String(sensor.a3);

  Serial.println(csvData);
  client.publish(mqtt_topic, csvData.c_str());
}

void setup() {
  Serial.begin(9600);
  connectWiFi();
  connectMQTT();
}

void loop() {
  readSensor();
  sendCSV();
  delay(10000); // Executa a cada 10 segundos
}

```
