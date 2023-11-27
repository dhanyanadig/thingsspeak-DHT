#include <WiFi.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include "ThingSpeak.h"

const int DHT_PIN = 15;
const int LED_PIN = 13;
const char* WIFI_NAME = "Wokwi-GUEST";
const char* WIFI_PASSWORD = "";
const int myChannelNumber = 2340624;
const char* myApiKey = "88ER89ODZQ2U5Y9D";
const char* server = "api.thingspeak.com";

DHT dhtSensor(DHT_PIN, DHT22); 
WiFiClient client;

void setup() {
  Serial.begin(115200);
  dhtSensor.begin();
  pinMode(LED_PIN, OUTPUT);
  
  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Wifi not connected");
  }
  
  Serial.println("Wifi connected !");
  Serial.println("Local IP: " + String(WiFi.localIP()));
  WiFi.mode(WIFI_STA);
  ThingSpeak.begin(client);
}

void loop() {
  float humidity = dhtSensor.readHumidity();
  float temperature = dhtSensor.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  ThingSpeak.setField(1, temperature);
  ThingSpeak.setField(2, humidity);

  if (temperature > 35 ) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }

  int x = ThingSpeak.writeFields(myChannelNumber, myApiKey);

  Serial.println("Temp: " + String(temperature, 2) + "°C");
  Serial.println("Humidity: " + String(humidity, 1) + "%");

  if (x == 200) {
    Serial.println("Data pushed successfully");
  } else {
    Serial.println("Push error: " + String(x));
  }

  Serial.println("---");
  delay(5000);
}

