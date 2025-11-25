# Mapadecalor

Este projeto realiza a coleta contínua da potência do sinal WiFi (RSSI) em dBm utilizando um ESP32, exibindo os valores na porta serial da Arduino IDE e enviando-os para uma plataforma IoT via MQTT.
Na plataforma, os dados são exibidos em um gráfico em tempo real (tempo × dBm).

Além disso, são realizados testes de variação de sinal em diferentes ambientes, incluindo a simulação de uma gaiola de Faraday no elevador do Inteli.

# Hardware Utilizado

 - ESP32 (modelo com WiFi)
 - Cabo USB
 - Antena Wifi para Módulo Esp8266

# Código MQTT

```cpp
#include "UbidotsEsp32Mqtt.h"
const char *WIFI_SSID = "Inteli.Iot"; 
const char *WIFI_PASS = "%(Yk(sxGMtvFEs.3"; 
const char *UBIDOTS_TOKEN = "BBUS-aRsK502Es4rtfx4QdEWYn1778hzLAs"; 
const char *DEVICE_LABEL = "antena_ll"; 
const char *VARIABLE_LABEL = "dbm";
const char *CLIENT_ID = "6925baefd67daeef2459c3df"; 
Ubidots ubidots(UBIDOTS_TOKEN, CLIENT_ID); 
const int PUBLISH_FREQUENCY = 3000; 
unsigned long timer;
unsigned long last_publish = 0;
const unsigned long PUBLISH_INTERVAL = PUBLISH_FREQUENCY;


void callback(char *topic, byte *payload, unsigned int length){
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++){
    Serial.print((char)payload[i]);
  }
  Serial.println();
}
float value1;
void setup(){
  Serial.begin(115200);
  ubidots.setDebug(true); 
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  timer = millis();
}
void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    int32_t dBm = WiFi.RSSI();
    if (millis() - last_publish > PUBLISH_INTERVAL) {
      ubidots.add(VARIABLE_LABEL, dBm);
      ubidots.publish(DEVICE_LABEL);
 	Serial.printf("Nível de Sinal Wi-Fi: %d dBm\n", dBm);
      last_publish = millis();
    }
  } else {
    Serial.println("Wi-Fi desconectado! Tentando reconectar...");
    WiFi.begin(WIFI_SSID, WIFI_PASS);
  }
}
```

# Conclusão

Durante o desenvolvimento da tarefa, todo o processo de coleta e publicação dos valores de dBm funcionou conforme o planejado, porém alguns obstáculos impactaram o andamento. Um dos principais problemas foi a porta COM, que em diversos momentos deixou de reconhecer o ESP32, exigindo reinicializações e troca temporária de porta para que o upload pudesse ser feito corretamente. Além disso, houve instabilidade na rede WiFi em algumas regiões da Inteli, o que ocasionou quedas momentâneas na conexão do ESP32 e interrupções na transmissão MQTT. No entanto esses problemas foram resolvidos para que as coletas de dados serem realizadas com êxito.
