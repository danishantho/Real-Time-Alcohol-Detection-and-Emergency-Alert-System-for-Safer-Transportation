# Real-Time-Alcohol-Detection-and-Emergency-Alert-System-for-Safer-Transportation
Code Project Description
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>
#include <TinyGPS++.h>

/* ---------- LCD ---------- */
LiquidCrystal_I2C lcd(0x27, 16, 2);

/* ---------- GSM & GPS ---------- */
SoftwareSerial gsm(2, 3);
SoftwareSerial gpsSerial(4, 5);
TinyGPSPlus gps;

/* ---------- Pins ---------- */
#define MQ3 A0
#define LED 7

/* ---------- Variables ---------- */
int alcoholThreshold = 500;
bool alertSent = false;

float latitude = 0.0;
float longitude = 0.0;

/* ---------- Police Station Numbers ---------- */
String policeStationNumber = "+91XXXXXXXXXX"; // default

String police1 = "+91AAAAAAAAAA";
String police2 = "+91BBBBBBBBBB";
String police3 = "+91CCCCCCCCCC";

/* ---------- Setup ---------- */
void setup() {

  pinMode(LED, OUTPUT);
  digitalWrite(LED, LOW);

  Serial.begin(9600);
  gsm.begin(9600);
  gpsSerial.begin(9600);

  lcd.init();
  lcd.backlight();

  lcd.setCursor(0,0);
  lcd.print("Alcohol Detector");
  lcd.setCursor(0,1);
  lcd.print("System Ready");

  delay(2000);
  lcd.clear();
}

/* ---------- Main Loop ---------- */
void loop() {

  readGPS();   // update GPS location

  int alcoholValue = analogRead(MQ3);

  Serial.print("Alcohol: ");
  Serial.println(alcoholValue);

  lcd.setCursor(0,0);
  lcd.print("Alcohol:");
  lcd.print(alcoholValue);
  lcd.print("   ");

  if (alcoholValue > alcoholThreshold) {

    digitalWrite(LED, HIGH);

    lcd.setCursor(0,1);
    lcd.print("ALCOHOL ALERT!");

    if (!alertSent) {

      selectPoliceStation();  // NEW FUNCTION
      sendLocationSMS();

      alertSent = true;
    }

  }
  else {

    digitalWrite(LED, LOW);

    lcd.setCursor(0,1);
    lcd.print("Safe to Drive ");

    alertSent = false;
  }

  delay(1000);
}

/* ---------- GPS Function ---------- */
void readGPS() {

  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  if (gps.location.isValid()) {

    latitude = gps.location.lat();
    longitude = gps.location.lng();

    Serial.print("Lat: ");
    Serial.print(latitude,6);

    Serial.print(" Lon: ");
    Serial.println(longitude,6);
  }
}

/* ---------- Zone Based Police Selection ---------- */
void selectPoliceStation() {

  if(latitude >= 13.00 && latitude <= 13.10) {
      policeStationNumber = police1;
  }

  else if(latitude > 13.10 && latitude <= 13.20) {
      policeStationNumber = police2;
  }

  else {
      policeStationNumber = police3;
  }

}

/* ---------- SMS Function ---------- */
void sendLocationSMS() {

  lcd.clear();
  lcd.print("Sending SMS...");

  gsm.println("AT");
  delay(500);

  gsm.println("AT+CMGF=1");
  delay(500);

  gsm.print("AT+CMGS=\"");
  gsm.print(policeStationNumber);
  gsm.println("\"");

  delay(500);

  gsm.println("ALERT!");
  gsm.println("Alcohol Detected.");

  if(gps.location.isValid()) {

    gsm.print("Location: https://maps.google.com/?q=");
    gsm.print(latitude,6);
    gsm.print(",");
    gsm.println(longitude,6);

  } else {

    gsm.println("Location not available.");
  }

  gsm.write(26); // CTRL+Z
  delay(3000);

  lcd.clear();
  lcd.print("SMS Sent");
  delay(2000);
  lcd.clear();
}
