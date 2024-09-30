# PressureSensor-sprint
Este projeto implementa um sensor de pressão para pneus utilizando um módulo HX711 conectado a um microcontrolador ESP32. O sensor mede a pressão (ou peso) e publica os dados em um broker MQTT. Além disso, o sistema inclui um LED e um buzzer que são ativados quando a pressão ultrapassa um determinado limite.

## Funcionalidades
Medição de peso utilizando o módulo HX711;
Conexão Wi-Fi para enviar dados via MQTT;
Publicação de dados em formato JSON no broker MQTT;
Indicação visual e sonora (LED e buzzer) quando a pressão ultrapassa o limite.

## Componentes Necessários
| Componente    | Quantidade    |
| ------------- | ------------- |
| ESP32  | 1 |
| LED  | 1 
| Buzzer | 1  |
| HX711 | 1 |
| Led amarelo  | 1  |
| Buzzer  | 1  |
| Célula de carga| 1  |

## Dependências Code
Certifique-se de ter as seguintes bibliotecas instaladas:

HX711
WiFi
PubSubClient
Variáveis Configuráveis
SSID: Nome da rede Wi-Fi.
PASSWORD: Senha da rede Wi-Fi (deixe vazio se não houver senha).
mqtt_server: Endereço IP do broker MQTT.
mqtt_port: Porta do broker MQTT (geralmente 1883).
mqtt_topic: Tópico MQTT para publicação dos dados.
threshold: Valor limite de peso (em kg) para acionar o LED e o buzzer.

## Code Compilation
Abra o Arduino IDE ou outro ambiente de desenvolvimento compatível com ESP32.
Copie o código fornecido e cole na IDE.
Atualize as variáveis de configuração conforme necessário.
Selecione a placa ESP32 e a porta correta.
Carregue o código para o ESP32.

## Monitoramento
Abra o monitor serial (115200 bps) para visualizar as leituras de peso e o status das conexões.
Os dados de peso serão publicados no broker MQTT conforme a configuração.


## Referências

- [Documentação oficial do ESP32](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/index.html)
- [Documentação oficial da AZURE](https://github.com/MicrosoftDocs/azure-docs)

## Autores
Projeto desenvolvido para a matéria de Edge Computing do Professor Fabio Cabrini, por: 
Felipe Genistretti Rodrigues;
Nicolle Pellegrino Jelinski;
Nicolas Aquino Borges;
Renan Simões Gonçalves;
Vitor Rivas Cardoso.

# Code
```
#include "HX711.h"  // Inclui a biblioteca HX711
#include <WiFi.h>   // Inclui a biblioteca WiFi
#include <PubSubClient.h>  // Inclui a biblioteca PubSubClient

// Configuração dos pinos usados pelo HX711
const int loadCellDoutPin = 4;  // Pino de dados do HX711
const int loadCellSckPin = 5;   // Pino de clock do HX711

// Configuração dos pinos para LED e Buzzer
const int ledPin = 12;    // Pino onde o LED está conectado
const int buzzerPin = 13; // Pino onde o buzzer está conectado

// Definição do valor limite de peso (30 kg)
const float threshold = 3000.0; // Limite de peso (3000 unidades)

// Configurações - variáveis editáveis
const char* SSID = "Wokwi-GUEST";               // Nome da rede Wi-Fi
const char* PASSWORD = "";                      // Senha da rede Wi-Fi
const char* mqtt_server = "20.220.173.42";      // Endereço do broker MQTT
const int mqtt_port = 1883;                     // Porta do broker MQTT
const char* mqtt_topic = "/TEF/sensor001/attrs"; // Tópico para publicação MQTT

HX711 scale;  // Objeto da classe HX711
WiFiClient espClient;  // Cliente WiFi
PubSubClient client(espClient);  // Cliente MQTT

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando ao WiFi ");
  Serial.println(SSID);
  
  WiFi.begin(SSID, PASSWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi conectado!");
  Serial.println("Endereço IP: ");
  Serial.println(WiFi.localIP());
}

void reconnect() {
  // Loop até conseguir se reconectar ao MQTT
  while (!client.connected()) {
    Serial.print("Tentando se conectar ao MQTT...");
    // Tenta se conectar ao broker MQTT
    if (client.connect("ESP32Client")) {
      Serial.println("Conectado ao broker MQTT");
    } else {
      Serial.print("Falha na conexão, código de erro: ");
      Serial.print(client.state());
      delay(2000); // Aguarda 2 segundos antes de tentar novamente
    }
  }
}

void setup() {
  Serial.begin(115200);
  
  // Inicializa o HX711
  scale.begin(loadCellDoutPin, loadCellSckPin);
  
  // Inicializa pinos para LED e Buzzer
  pinMode(ledPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  digitalWrite(buzzerPin, LOW);

  // Conecta ao Wi-Fi
  setup_wifi();
  
  // Configura o broker MQTT
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  if (scale.is_ready()) {
    float weight = scale.get_units();
    Serial.print("Peso: ");
    Serial.print(weight);
    Serial.println(" kg");

    // Prepara os dados para envio no formato JSON
    String payload = "{\"id\": \"Sensor1\", \"type\": \"Sensor\", \"pressure\": {\"value\": " + String(weight) + ", \"type\": \"Number\"}}";
    char attributes[200];
    payload.toCharArray(attributes, 200);

    // Publica os dados no broker MQTT
    if (client.publish(mqtt_topic, attributes)) {
      Serial.println("Dados publicados no MQTT");
    } else {
      Serial.println("Falha ao publicar dados no MQTT");
    }

    // Lógica para acionar LED e Buzzer
    if (weight > threshold) {
      digitalWrite(ledPin, HIGH);  // Liga o LED
      tone(buzzerPin, 1000);       // Ativa o Buzzer
    } else {
      digitalWrite(ledPin, LOW);   // Desliga o LED
      noTone(buzzerPin);           // Desativa o Buzzer
    }
  } else {
    Serial.println("HX711 não está pronto.");
  }

  delay(1000); // Aguarda 1 segundo antes de repetir
}
```

