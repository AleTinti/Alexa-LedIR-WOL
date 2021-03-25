#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <WebSocketsClient.h>
#include <ArduinoJson.h>
#include <IRsend.h>
#include <WiFiUdp.h>
#include <WakeOnLan.h>

WiFiUDP UDP;
WakeOnLan WOL(UDP);
ESP8266WiFiMulti WiFiMulti;
WebSocketsClient webSocket;
WiFiClient client;
IRsend irsend(D5);

#define MyApiKey "SINRINC API KEY"
#define MySSID "SSID" //SSID
#define MyWifiPassword "PASSWORD"

#define API_ENDPOINT "http://sinric.com"
#define HEARTBEAT_INTERVAL 300000

uint64_t heartbeatTimestamp = 0;
bool isConnected = false;

void setup() {
  Serial.begin(115200);

  WOL.setRepeat(3, 100);
  WOL.calculateBroadcastAddress(WiFi.localIP(), WiFi.subnetMask());
  irsend.begin();
  digitalWrite(BUILTIN_LED, LOW);

  WiFiMulti.addAP(MySSID, MyWifiPassword);
  Serial.println();
  Serial.print("Connecting to WiFI: ");
  Serial.println(MySSID);

  while(WiFiMulti.run() != WL_CONNECTED) {
    delay(250);
    Serial.print("-");
  }

  if(WiFiMulti.run() == WL_CONNECTED) {
    Serial.println("");
    Serial.print("WiFi connected ");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    digitalWrite(BUILTIN_LED, LOW);
  }

  webSocket.begin("iot.sinric.com", 80, "/");
  webSocket.onEvent(webSocketEvent);
  webSocket.setAuthorization("apikey", MyApiKey);
  webSocket.setReconnectInterval(5000);
}

void loop() {
  webSocket.loop();
  digitalWrite(BUILTIN_LED, LOW);
  if(isConnected) {
      uint64_t now = millis();

      if((now - heartbeatTimestamp) > HEARTBEAT_INTERVAL) {
          heartbeatTimestamp = now;
          webSocket.sendTXT("H");
      }
  }
}

void webSocketEvent(WStype_t type, uint8_t * payload, size_t length) {
  switch(type) {
    case WStype_DISCONNECTED:
      isConnected = false;
      Serial.printf("Webservice disconnected");
      break;
    case WStype_CONNECTED: {
      isConnected = true;
      Serial.printf("Connected to sinric");
      Serial.printf("Waiting for commands");
      }
      break;
    case WStype_TEXT: {
        Serial.printf("Get text: %s\n", payload);

#if ARDUINOJSON_VERSION_MAJOR == 5
        DynamicJsonBuffer jsonBuffer;
        JsonObject& json = jsonBuffer.parseObject((char*)payload);
#endif

#if ARDUINOJSON_VERSION_MAJOR == 6
        DynamicJsonDocument json(1024);
        deserializeJson(json, (char*) payload);
#endif
        String deviceId = json ["deviceId"];
        String action = json ["action"];

        if(action == "setPowerState") {
            String value = json ["value"];
            if(value == "ON") {
                turnOn(deviceId);
            } else {
                turnOff(deviceId);
            }
        }
      }
      break;
    default: break;
  }
}

void turnOn(String deviceId) {
  if (deviceId == "SINRINC DEVICE ID")
  {
    irsend.sendNEC(16236607, 32); //ON
  }

  if (deviceId == "") //Device ID
  {
    WOL.sendMagicPacket("MAC ADDRESS", 9);
  }
}

void turnOff(String deviceId) {
   if (deviceId == "SINRINC DEVICE ID")
   {
     irsend.sendNEC(16203967, 32); //OFF
   }
}
