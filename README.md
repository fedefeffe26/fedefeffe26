/*
   MQTT Exmple for SeeedStudio Wio Terminal
   Author: Salman Faris
   Date: 31/07/2020
   Last Updates: 02/08/2020

   MQTT Broker broker.mqtt-dashboard.com
   Subscribe Topic Name: WTIn
   Publish Topic Name: WTout
  - publishes "hello world" to the topic "WTout"
  - subscribes to the topic "WTin", printing out any messages

*/


#include "rpcWiFi.h"
#include"TFT_eSPI.h"
#include <PubSubClient.h>


// Update these with values suitable for your network.
const char* ssid = "EOLO_023890"; // WiFi Name
const char* password = "2zTzELRew";  // WiFi Password
const char* mqtt_server = "broker.hivemq.com";  // MQTT Broker URL

TFT_eSPI tft;

WiFiClient wioClient;
PubSubClient client(wioClient);
long lastMsg = 0;
char msg[50];
int value = 0;


#include"LIS3DHTR.h"

LIS3DHTR<TwoWire> lis;

#include <Arduino_JSON.h>

void setup_wifi() {

  delay(10);

  tft.setTextSize(2);
  tft.setCursor((320 - tft.textWidth("Connecting to Wi-Fi..")) / 2, 120);
  tft.print("Connecting to Wi-Fi..");

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password); // Connecting WiFi

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");

  tft.fillScreen(TFT_BLACK);
  tft.setCursor((320 - tft.textWidth("Connected!")) / 2, 120);
  tft.print("Connected!");

  Serial.println("IP address: ");
  Serial.println(WiFi.localIP()); // Display Local IP Address
  tft.fillScreen(TFT_WHITE);
}

void callback(char* topic, byte* payload, unsigned int length) {
  //tft.fillScreen(TFT_BLACK);
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  char buff_p[length];
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
    buff_p[i] = (char)payload[i];
  }
  Serial.println();
  buff_p[length] = '\0';
  String msg_p = String(buff_p);
  tft.fillScreen(TFT_BLACK);
  tft.setCursor((320 - tft.textWidth("MQTT Message")) / 2, 90);
  tft.print(" " );
  tft.setCursor((320 - tft.textWidth(msg_p)) / 2, 120);
  tft.print(msg_p); // Print receved payload
  // Wait 5 seconds before retrying
  delay(5000);tft.fillScreen(TFT_WHITE);
}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "WioTerminal-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("/ferreromortiannoR", "hello world");
      // ... and resubscribe
      client.subscribe("/ferreromortianno2");
      if (digitalRead(WIO_KEY_A) == LOW);{
      client.subscribe("/ferreromortianno");}
    } 
    else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {


  pinMode(WIO_KEY_A, INPUT_PULLUP);
  pinMode(WIO_KEY_B, INPUT_PULLUP);
  pinMode(WIO_KEY_C, INPUT_PULLUP);

  lis.begin(Wire1);
  lis.setOutputDataRate(LIS3DHTR_DATARATE_100HZ);
  lis.setFullScaleRange(LIS3DHTR_RANGE_4G);

  pinMode(WIO_LIGHT, INPUT);

  tft.begin();
  tft.fillScreen(TFT_BLACK);
  tft.setRotation(3);


  Serial.println();
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);// Connect the MQTT Server
  client.setCallback(callback);
}

void loop() {


  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  JSONVar pippo;

  pippo["x"] = lis.getAccelerationX();
  pippo["y"] = lis.getAccelerationY();
  pippo["z"] = lis.getAccelerationZ();
  pippo["l"] = analogRead(WIO_LIGHT);

  String jsonString = JSON.stringify(pippo);

  long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    ++value;

    // put your main code here, to run repeatedly:
    if (digitalRead(WIO_KEY_A) == LOW) {
      Serial.println("A Key pressed");
      client.publish("/ferreromortiannoR","A");
      }

    if (digitalRead(WIO_KEY_B) == LOW) {
      Serial.println("B Key pressed");
       client.publish("/ferreromortianno", "B");
       client.subscribe("/ferreromortianno2");
    }
    if (digitalRead(WIO_KEY_C) == LOW) {
      Serial.println("C Key pressed");
       client.publish("/ferreromortianno", "C");
       client.subscribe("/ferreromortianno2");
    }
    delay(200);


    //    char text = jsonString;
    //   snprintf (text, 50, "Wio Terminal #%ld", value);
    //   Serial.print("Publish message: ");
    //   Serial.println(jsonString);

    //   const char* test = jsonString;

    //  client.publish("WTout", test);
  }
}
