#define BLYNK_TEMPLATE_ID "TMPL6UB-NrV4u"
#define BLYNK_TEMPLATE_NAME "Smart Parking"
#define BLYNK_AUTH_TOKEN "gcGRlyOVzTwFGObs7P7zD7HKb9IVZlg3"
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <LiquidCrystal_I2C.h>
#include <ESP32Servo.h>

char ssid[] = "Tang 1 2"; 
char pass[] = "12345678";

#define PIN_SERVO 18
#define PIN_SENSOR_IN 4
#define PIN_SENSOR_OUT 5

LiquidCrystal_I2C lcd(0x27, 16, 2);

Servo gateServo;

int slots = 5;
int lastSlots = -1;

void updateSystem() {
  if (slots == lastSlots) return;
  lastSlots = slots;

  lcd.clear();

  if (slots <= 0) {
    lcd.setCursor(0, 0);
    lcd.print("    HET CHO!   ");

    lcd.setCursor(0, 1);
    lcd.print(" VUI LONG RA   ");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("   BAI DO XE   ");

    lcd.setCursor(0, 1);
    lcd.print("TRONG: ");
    lcd.print(slots);
    lcd.print("       "); 
  }

  Blynk.virtualWrite(V1, slots);
}

void openGate() {
  gateServo.write(90);
  delay(2000); 
  gateServo.write(0);
}

void setup() {
  Serial.begin(115200);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("DANG KET NOI...");

  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(300);
    Serial.print(".");
  }

  Blynk.config(BLYNK_AUTH_TOKEN);
  Blynk.connect();

  pinMode(PIN_SENSOR_IN, INPUT_PULLUP);
  pinMode(PIN_SENSOR_OUT, INPUT_PULLUP);

  gateServo.attach(PIN_SERVO, 500, 2400);
  gateServo.write(0);

  updateSystem();
}

void loop() {
  Blynk.run();

  if (digitalRead(PIN_SENSOR_IN) == LOW) {
    if (slots > 0) {
      openGate();
      slots--;
      updateSystem();
    }
    while (digitalRead(PIN_SENSOR_IN) == LOW) delay(10);
  }

  if (digitalRead(PIN_SENSOR_OUT) == LOW) {
    if (slots < 5) {
      openGate();
      slots++;
      updateSystem();
    }
    while (digitalRead(PIN_SENSOR_OUT) == LOW) delay(10);
  }
}
