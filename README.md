## CESAR AUGUSTO GONZALEZ INFANTE
## SENSOR ULTRASONICO Y DHT22 CON NODE RED
## Material Necesario

Para realizar esta practica se usaran los siguientes elementos:

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor Ultrasonico
- Sensor DHT22
- [NODERED](http://localhost:1880/)



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;



const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 13;   //Pin digital 3 para el Echo del sensor
  long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros


// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="CESARGONZALEZ";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);

    pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = String(d);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("CESARGONZALE3", output.c_str());
  }


  


  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
  Serial.print("Temperatura: ");
  Serial.print(data.temperature);      //Enviamos serialmente el valor de la distancia
  Serial.print("°C");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
  Serial.print("Humedad: ");
  Serial.print(data.humidity);      //Enviamos serialmente el valor de la distancia
  Serial.print("%");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms

}



```

2. Instalar las siguientes librerias:

![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG2.png?raw=true)

### Instrucciónes de operación

1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
3. Colocar distancia a marcar dando *doble click* al sensor **Ultrasónico** 
4. Colocar temperatura y humedad a marcar dando *doble click* al sensor **DHT22** 
5. Abrir Simbolo de sistema como administrador tras instalar el programa node red y escribir  node-red en la terminal del sistema tras esto ir a un navegador web y escribir: (http://localhost:1880/) y estaremos en node red.

5. Juntar los elementos siguientes mostrados en la pantalla:

![](https://github.com/CesarG16/SENSORSONICORED/blob/main/IMAG1.png?raw=true)

6. Los elementos puestos son : mqtt in, json, function, function, function, gauge, gauge, gauge y chart.
7. Después abrir Dashboard en la esquina superior de la derecha como se ve en la siguiente imagen:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE4.png?raw=true)

8. Dar click en +tab.
9. Dar click en +group y añadir dos. 
10. De alli tanto en tab como en los grupos se les da en los botones de editar de cada uno y en la seccion de name se le asigna un nombre a cada uno.
11. de alli se da click a cada uno de los elementos unidos anteriormente y se editan empezando por el primero, hasta el ultimo como se ven en las siguientes imágenes:

![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG4.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG5.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG6.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG7.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG8.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG9.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG10.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG11.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG12.png?raw=true)


12. Tras esto se da en el boton rojo de arriba a la derecha que dice Deploy y cuando esté listo se le da en el siguiente boton que esta abajo de Deploy para que abra una nueva página y se visualice el resultado:

![](https://github.com/CesarG16/DHT22RED/blob/main/EJE12.png?raw=true)

## Resultados


Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen y en la página creada por node - red.


![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG2.png?raw=true)
![](https://github.com/CesarG16/Sensorultrasonicoydht22red/blob/main/IMG13.png?raw=true)




## Evidencias

[Página](https://wokwi.com/projects/367824656696140801)


# Créditos

Desarrollado por Ing. Cesar Augusto Gonzalez Infante

- [GitHub](https://github.com/CesarG16)