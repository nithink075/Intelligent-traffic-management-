#include <WiFi.h>

// Wi-Fi Details
const char* WIFI_SSID = "keerthana";
const char* WIFI_PASSWORD = "keer@1234";

// IR Sensor Pins
#define IR1 13
#define IR2 14
#define IR3 27
#define IR4 26
#define IR5 25
#define IR6 33

// Single Traffic Signal LED Pins
#define RED_LED 16
#define YELLOW_LED 17
#define GREEN_LED 18

// Signal Timing
const unsigned long LOW_GREEN = 5000;
const unsigned long MEDIUM_GREEN = 7000;
const unsigned long HIGH_GREEN = 10000;
const unsigned long YELLOW_TIME = 2000;

// Traffic Count
int vehicleCount = 0;

// Wi-Fi Connection
void connectWiFi() {
  Serial.println("Connecting to Wi-Fi...");

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  unsigned long startTime = millis();

  while (WiFi.status() != WL_CONNECTED &&
         millis() - startTime < 20000) {
    delay(500);
    Serial.print(".");
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\nWi-Fi Connected!");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("\nWi-Fi connection failed.");
    Serial.println("Continuing offline.");
  }
}

// Turn all LEDs OFF
void allOff() {
  digitalWrite(RED_LED, LOW);
  digitalWrite(YELLOW_LED, LOW);
  digitalWrite(GREEN_LED, LOW);
}

// Red Signal
void redSignal() {
  allOff();
  digitalWrite(RED_LED, HIGH);
  Serial.println("TRAFFIC SIGNAL: RED");
}

// Yellow Signal
void yellowSignal() {
  allOff();
  digitalWrite(YELLOW_LED, HIGH);
  Serial.println("TRAFFIC SIGNAL: YELLOW");
  delay(YELLOW_TIME);
}

// Green Signal
void greenSignal(unsigned long duration) {
  allOff();
  digitalWrite(GREEN_LED, HIGH);
  Serial.println("TRAFFIC SIGNAL: GREEN");

  Serial.print("Green Duration: ");
  Serial.print(duration / 1000);
  Serial.println(" seconds");

  delay(duration);
}

// Setup
void setup() {
  Serial.begin(115200);

  pinMode(IR1, INPUT);
  pinMode(IR2, INPUT);
  pinMode(IR3, INPUT);
  pinMode(IR4, INPUT);
  pinMode(IR5, INPUT);
  pinMode(IR6, INPUT);

  pinMode(RED_LED, OUTPUT);
  pinMode(YELLOW_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);

  redSignal();

  Serial.println("INTELLIGENT TRAFFIC MONITORING SYSTEM");

  connectWiFi();
}

// Main Loop
void loop() {
  // Read all 6 IR sensors
  int s1 = digitalRead(IR1);
  int s2 = digitalRead(IR2);
  int s3 = digitalRead(IR3);
  int s4 = digitalRead(IR4);
  int s5 = digitalRead(IR5);
  int s6 = digitalRead(IR6);

  // Count occupied sensors (active LOW)
  vehicleCount = 0;

  if (s1 == LOW) vehicleCount++;
  if (s2 == LOW) vehicleCount++;
  if (s3 == LOW) vehicleCount++;
  if (s4 == LOW) vehicleCount++;
  if (s5 == LOW) vehicleCount++;
  if (s6 == LOW) vehicleCount++;

  Serial.println("\n-----------------------------");
  Serial.print("Occupied Sensors: ");
  Serial.println(vehicleCount);

  unsigned long greenDuration;
  String trafficLevel;

  // Determine traffic level
  if (vehicleCount >= 4) {
    trafficLevel = "HIGH";
    greenDuration = HIGH_GREEN;
  }
  else if (vehicleCount >= 2) {
    trafficLevel = "MEDIUM";
    greenDuration = MEDIUM_GREEN;
  }
  else {
    trafficLevel = "LOW";
    greenDuration = LOW_GREEN;
  }

  Serial.print("Traffic Level: ");
  Serial.println(trafficLevel);

  // Traffic signal sequence
  redSignal();
  delay(1000);

  greenSignal(greenDuration);

  yellowSignal();

  redSignal();
  delay(1000);
}
