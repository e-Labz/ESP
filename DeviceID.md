
Este código obtém um identificador único do ESP32 ou ESP8266 com base no endereço MAC da placa. Ele configura a comunicação serial, define o modo WiFi sem precisar conectar-se a uma rede e exibe o identificador único no monitor serial em formato hexadecimal. É útil para identificar dispositivos em projetos IoT.

`File: DeviceID.ino`
```
// Verifica se a placa é ESP32 ou ESP8266 e inclui a biblioteca WiFi correspondente
#ifdef ESP32
  #include <WiFi.h>         // Biblioteca para ESP32
#elif defined ESP8266
  #include <ESP8266WiFi.h>  // Biblioteca para ESP8266
#else
  #error Placa inválida     // Gera erro se a placa não for compatível
#endif

/*************************************
 * deviceID - Obtém um identificador único do ESP baseado no MAC
 *************************************/
uint64_t deviceID() {
  uint8_t mac[6];             // Array para armazenar o MAC
  WiFi.macAddress(mac);       // Obtém o endereço MAC do dispositivo
  uint64_t id = 0;            // Variável para armazenar o ID único
  for (int i = 0; i < 6; i++) {
    id = (id << 8) | mac[i];  // Converte os 6 bytes do MAC para um único número de 64 bits
  }
  return id;                  // Retorna o ID único do dispositivo
}

void setup() {
  // Inicializa a comunicação serial com velocidade apropriada para cada placa
#ifdef ESP32
  Serial.begin(115200); // Velocidade padrão para ESP32
#else
  Serial.begin(74480);  // Velocidade padrão para ESP8266
#endif

  WiFi.mode(WIFI_STA);  // Define o modo WiFi para obter o MAC sem necessidade de conexão
  delay(1000);          // Aguarda um pouco para garantir a leitura do ID corretamente

  // Exibe o ID único do dispositivo no monitor serial
  Serial.println();
  Serial.print(F("deviceID: ")); // Usa F() para economizar RAM no ESP8266
  Serial.println(deviceID(), HEX);  // Exibe o ID no formato hexadecimal
}

void loop() {
  // Nada a fazer, pois o objetivo do programa é apenas exibir o ID no setup()
}
```
