# Práctica ESP32 con Ultrasónico usando Node Red de Diego Llampallas
Este repositorio muestra como podemos programar una ESP32 con el sensor ultrasónico y ver los resultados en una página gráficados llamado node red.


## Introducción
A través de la página https://wokwi.com/  se puede hacer simulaciones de programas con arduino y sensores.
### Descripción

La ```Esp32``` la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (```Ultrasónico```) para medir distancia y a tráves de la página de node red se pueda visualizar los datos mostrados del sensor graficandolos usando gráficas interactivas que se mueven conforme sensea el sensor dht22 mostrandolos en una página web creada por node red; Cabe aclarar que esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/).


## Material Necesario

Para realizar esta practica se usaran los siguientes elementos:

- [WOKWI](https://https://wokwi.com/)
- Tarjeta ESP 32
- Sensor ultrasónico
- Pantalla LCD 16X2
- [NODE RED](http://localhost:1880/)


## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).

### Instrucciones de preparación de entorno 
1. Una vez dentro de wokwi seleccionar la tarjeta ESP32

![](https://github.com/DiegoLlampallas/Practica-DHT22/blob/main/6.png?raw=true)

2. Abrir la terminal de programación y colocar la siguente programación:

## Programación

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2

// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="DiegoLlampallas";
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

#include <LiquidCrystal_I2C.h>
#define I2C_ADDR    0x27
#define LCD_COLUMNS 20
#define LCD_LINES   4
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 13;   //Pin digital 3 para el Echo del sensor

const int DHT_PIN = 15;



LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);
void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);



  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
    lcd.init();
  lcd.backlight();
}





void loop()
{

digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  

delay(1000);

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
    doc["DISTANCIA"] =d;
   
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Diego/Distancia", output.c_str());
  }



       
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print(" cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms

  lcd.setCursor(0, 0);
  lcd.print("Distancia: " + String(d) +"cm  ");
   lcd.setCursor(0, 1);
  lcd.print("Diego Llampallas");
  delay(1000);          //Hacemos una pausa de 100ms
 
}

```


## Partes
1. ![](https://github.com/DiegoLlampallas/SENSORULTRASONICO/blob/main/5.png?raw=true)
2. ![](https://github.com/DiegoLlampallas/SENSORULTRASONICO/blob/main/6.png?raw=true)
3. ![](https://github.com/DiegoLlampallas/SENSORULTRASONICO/blob/main/7.png?raw=true)
4. ![](https://github.com/DiegoLlampallas/SENSORULTRASONICO/blob/main/8.png?raw=true)
## Librerias
![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/2.png?raw=true)

## Conexión

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/1.png?raw=true)

# Configuración y conexión de node red

## Conexión de node red
 1. Colocar bloque ```mqqtt in```.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/8.png?raw=true)

 2. Colocar bloque ```json```.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/9.png?raw=true)

 3. Colocar bloque ```function```.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/10.png?raw=true)

 4. Colocar bloque ```gauge```.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/11.png?raw=true)

 5. Colocar bloque ```chart```.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/12.png?raw=true)

 6. Conectamos todos los componentes como se muestra en la imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/3.png?raw=true)

Tras conectar todo se pasa a la configuración.

## Configuración de node red

1. Vamos a la esquina superior derecha y en la "flechita que apunta  hacia abajo seleccionamos y buscamos "Dashboard".

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/21.png?raw=true)

2. Le damos de"+ tab".

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/22.png?raw=true)

3. Le damos en "+ group" dos veces para crear dos grupos.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/23.png?raw=true)

4. Le damos en "edit" al "tab" creado.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/24.png?raw=true)

5. Llenamos los datos con la información tal y como aparece en la siguiente imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/4.png?raw=true)

6. Le damos en "edit" al "grou 1" creado.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/26.png?raw=true)

7. Llenamos los datos con la información tal y como aparece en la siguiente imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/4.png?raw=true)


8. Le damos en "edit" al "grou 2" creado.

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/28.png?raw=true)

9. Llenamos los datos con la información tal y como aparece en la siguiente imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/5.png?raw=true)

Tras completar todo esto vamos a ir seleccionando y dando doble click a cada bloque y llenarlo con la información necesaria, en este caso de izquierda a derecha:


1. Edit "mqtt" y completar los datos mostrados en la imagen:

 ![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/6.png?raw=true)

2. Edit "json" y completar los datos mostrados en la imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/7.png?raw=true)

3. Edit "function" y completar los datos mostrados en la imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/8.png?raw=true)


4. Edit "gauge" y completar los datos mostrados en la imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/9.png?raw=true)

5. Edit "chart" y completar los datos mostrados en la imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/10.png?raw=true)

## Nota
Se anexa el bloque debug sólo para visualizar que existe comunicación entre el "wokwi" y el "node red" como se muestra en la siguiente imagen:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/0.png?raw=true)

Tras completar todo esto puede empezar la simulación de operación tanto en "wokwi" como en el "node red" como se mostrará acontinuación: 

### Instrucciónes de operación

1. Iniciar simulador "wokwi": 


2. Una vez que conecte debe mostrará el acceso:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/11.png?raw=true)

3. Tras esto podemos dar en iniciar en "node red" como se muestra en la siguiente imagen:

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/30.png?raw=true)

3. Colocar la distancia y humedad dando *doble click* al sensor **ultrasónico** 

4. Para acceder a la información mandada por el "ESP32" hay dos opciones para acceder a la página web creada:

Opción 1:

1. Escribiendo el código "localhost/1880/ic" en la imagen en una nueva página limpia:

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/31.png?raw=true)

Opción 2:

2. Dando click en la esquina superior derecha por donde esta dashboard:

![](https://github.com/DiegoLlampallas/DHT22NODERED/blob/main/32.png?raw=true)

## Resultados

Cuando haya funcionado, verás los valores dentro del monitor serial en "wokwi" y en la página creada.

## Funcionamiento

1. Funcionamiento en "wokwi":

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/11.png?raw=true)

2. Funcionamiento en la página web creada:

![](https://github.com/DiegoLlampallas/ULTRASONICNODERED/blob/main/12.png?raw=true)

## Evidencias

[Página](https://wokwi.com/projects/367787340645579777)


# Créditos

Desarrollado por Ing. Diego Alberto Llampallas Vega

- [GitHub](https://github.com/DiegoLlampallas)