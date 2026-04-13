#include <ESP8266WiFi.h>

const char* ssid     = "Nice";
const char* password = "superbro";

const char* host = "api.thingspeak.com";  
const char* apiKey = "VDOWSWOQTDMI94VD";  // Your Write API Key

// Ultrasonic sensor pins
const int trigPin = 12;
const int echoPin = 14;

void setup() {
  Serial.begin(115200);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Connect to WiFi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
}

void loop() {
  long duration;
  float distance;

  // Trigger ultrasonic sensor
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Read echo time
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2.0; // Convert to cm

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Send data to ThingSpeak
  WiFiClient client;
  const int httpPort = 80;

  if (client.connect(host, httpPort)) {
    String url = String("/update?api_key=") + apiKey + "&field1=" + String(distance);

    client.print(String("GET ") + url + " HTTP/1.1\r\n" +
                 "Host: " + host + "\r\n" +
                 "Connection: close\r\n\r\n");

    Serial.println("Data sent to ThingSpeak");
  } else {
    Serial.println("Failed to connect to ThingSpeak");
  }

  delay(15000);  // Wait 15 seconds for next update
}
