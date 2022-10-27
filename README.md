# cliente-NodeMCU
#Este programa realiza una conexión desde NodeMCU hacia la DB por MQTT
```ruby

#include <OneWire.h>
#include <DallasTemperature.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const int sensor = 4;
float temp;
char * dtostrf(double __val, signed char __width, unsigned char __prec, char * __s);

const char* ssid = "";
const char* password =  "";
const char*mqtt_server = "192.168.1.70";


WiFiClient espClient;
PubSubClient client(espClient); // creamos una instancia parcialmete inicializada

OneWire oneWireObjeto(sensor);
DallasTemperature sensorDS18B20(&oneWireObjeto);

void inicializa(){
  pinMode(sensor, INPUT);  
  sensorDS18B20.begin();
  }

void reporta(){
  sensorDS18B20.requestTemperatures();
  temp = sensorDS18B20.getTempCByIndex(0);
  Serial.print("temperatura=");
  Serial.println(temp);
  delay(1000);
  }
void setup (){
Serial.begin (115200);
WiFi.begin(ssid, password);

while (WiFi.status() != WL_CONNECTED){
  delay (500);
  Serial.print(".");
}

Serial.println("Dispositivo conectado");
Serial.println("direccion IP:");
Serial.println(WiFi.localIP());

client.setServer (mqtt_server, 1883);
client.setCallback(callback);
inicializa();
}

void callback (char* topic, byte* payload, unsigned int length){
Serial.print("mensaje cargado [");
Serial.print(topic);
Serial.print("]");

for (int i=0; i<length; i++){
Serial.print((char)payload[i]);
}
Serial.println();
}

//reconectar al cliente mqtt
void reconnect(){
  while (!client.connected()){

    if (client.connect("testgs")){
    Serial.println("conectado");
    client.subscribe("sensor/temperatura");
  }else{
    Serial.print("fallo, reconectando ...");
    Serial.print(client.state());
    Serial.println ("volviendo a conectar ");
    delay (3000);
    }
  }
}


void loop(){
  if (!client.connected()){
    reconnect();
  }
  client.loop();
  reporta();
 
    char Btemperatura[7];
    dtostrf(temp, 6, 2, Btemperatura);
      client.publish("sensor/temperatura", Btemperatura);
      Serial.print("Temperatura");
      Serial.print(Btemperatura); 
      delay (6000);
  
  // Esto hay que poner en la raspberry:
  //   mosquitto_sub -h localhost -t sensor/boton/luminaria1
  }
  ```
