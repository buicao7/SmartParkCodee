#define BLYNK_TEMPLATE_ID "TMPL6UB-NrV4u"
#define BLYNK_TEMPLATE_NAME "Smart Parking"
#define BLYNK_AUTH_TOKEN "gcGRlyOVzTwFGObs7P7zD7HKb9IVZlg3"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <LiquidCrystal_I2C.h>
#include <ESP32Servo.h>

char ssid[] = "Tang 1 2"; 
char pass[] = "12345678";

// ====== PIN ======
#define PIN_SERVO 18
#define PIN_SENSOR_IN 4
#define PIN_SENSOR_OUT 5

// ====== LCD ======
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ====== SERVO ======
Servo gateServo;

// ====== BIẾN ======
int slots = 5;
int lastSlots = -1;

// ================= UPDATE LCD =================
void updateSystem() {
  if (slots == lastSlots) return;
  lastSlots = slots;

  lcd.clear(); // 🔥 FIX QUAN TRỌNG: xóa toàn bộ màn hình

  if (slots <= 0) {
    lcd.setCursor(0, 0);
    lcd.print("    HET CHO!   "); // đủ 16 ký tự

    lcd.setCursor(0, 1);
    lcd.print(" VUI LONG RA   ");
  } else {
    lcd.setCursor(0, 0);
    lcd.print("   BAI DO XE   ");

    lcd.setCursor(0, 1);
    lcd.print("TRONG: ");
    lcd.print(slots);
    lcd.print("       "); // xóa dư
  }

  Blynk.virtualWrite(V1, slots);
}

// ================= MỞ CỔNG =================
void openGate() {
  gateServo.write(90);
  delay(2000); // đơn giản hóa cho ổn định
  gateServo.write(0);
}

// ================= SETUP =================
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

// ================= LOOP =================
void loop() {
  Blynk.run();

  // ===== XE VÀO =====
  if (digitalRead(PIN_SENSOR_IN) == LOW) {
    if (slots > 0) {
      openGate();
      slots--;
      updateSystem();
    }

    while (digitalRead(PIN_SENSOR_IN) == LOW) delay(10);
  }

  // ===== XE RA =====
  if (digitalRead(PIN_SENSOR_OUT) == LOW) {
    if (slots < 5) {
      openGate();
      slots++;
      updateSystem();
    }

    while (digitalRead(PIN_SENSOR_OUT) == LOW) delay(10);
  }
}
