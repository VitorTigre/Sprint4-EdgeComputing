# Sprint4-EdgeComputing
 👥 Integrantes

//Vitor Rodrigues Tigre – RM: 561746 //Josué Faria da Silva – RM: 563819 //Mariana Silva Oliveira – RM: 564241 //Jonas Esteves França – RM: 564143 //Augusto Valério – RM: 562185

📖 Descrição do Projeto
Este projeto tem como objetivo simular a coleta e o envio de dados biométricos utilizando um ESP32 integrado à plataforma ThingSpeak para armazenamento e visualização.
Os dados simulados são: Frequência cardíaca (bpm) Nível de oxigênio no sangue (%)
Aceleração nos eixos X, Y e Z
O ESP32 gera valores aleatórios para cada parâmetro e os envia periodicamente ao ThingSpeak através de requisições HTTP.

🏗️ Arquitetura Proposta
O ESP32 conecta-se à rede Wi-Fi configurada no código.
Os dados (batimentos, oxigênio e aceleração) são gerados de forma simulada utilizando a função random().
Esses dados são enviados para o ThingSpeak através de uma requisição HTTP.
O ThingSpeak armazena os valores nos respectivos campos do canal, permitindo sua visualização em gráficos.

🔧 Recursos Necessários
Placa ESP32
Acesso à internet (Wi-Fi)
Conta no ThingSpeak e chave de API para envio dos dados
(Obs: Os sensores de batimento, oxigênio e acelerômetro são simulados no código, não sendo necessário hardware adicional.)

🔌 Conexão dos sensores ao ESP32
Sensor / Componente	Pino ESP32
Potenciômetro (sinal)	GPIO 34
Acelerômetro SDA	GPIO 21
Acelerômetro SCL	GPIO 22
VCC	3.3V
GND	GND

💻 Código do ESP32
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "SEU_WIFI";
const char* password = "SENHA_WIFI";
const char* mqtt_server = "seu-endpoint-ats.iot.us-east-1.amazonaws.com";

WiFiClientSecure espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);

  espClient.setCACert(AmazonRootCA1);
  espClient.setCertificate(DeviceCert);
  espClient.setPrivateKey(DevicePrivateKey);

  client.setServer(mqtt_server, 8883);
  while (!client.connect("ESP32Device")) delay(1000);
}

void loop() {
  int potValue = analogRead(34);
  String msg = "{\"potenciometro\":" + String(potValue) + "}";
  client.publish("esp32/sensores", msg.c_str());
  delay(2000);
}

🌐 Teste via Postman
1. Configure a requisição
Método: POST
URL:
https://<endpoint>-ats.iot.us-east-1.amazonaws.com/topics/esp32/sensores
(Substitua <endpoint> pelo seu do IoT Core)

2. Aba “Authorization”
Type: AWS Signature
Access Key: (access key)
Secret Key: (secret key)
AWS Region: us-east-1
Service Name: iotdata
Add authorization data to: Request Headers

3. Aba “Body”
Selecione raw e JSON:
{
  "potenciometro": 512,
  "acelerometro": {"x": 0.15, "y": 0.10, "z": 9.80}
}

4. Enviar requisição
Clique em Send → você deve receber resposta 200 OK.

📡 Visualizar mensagens no AWS IoT Core
Vá até AWS IoT Core → Test → MQTT test client
Clique em Subscribe to a topic
Tópico: esp32/sensores
Veja as mensagens chegando em tempo real 🚀

✅ Resultado esperado
O ESP32 envia leituras reais dos sensores a cada 2 segundos.
As mensagens aparecem no MQTT Test Client da AWS.
É possível enviar dados de teste via Postman usando credenciais IAM.
