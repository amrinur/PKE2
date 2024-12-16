#define BLYNK_TEMPLATE_ID "TMPL66bPa9Gp7"
#define BLYNK_TEMPLATE_NAME "Smart Irrigation"
#define BLYNK_AUTH_TOKEN "hCupO-P2JYqypyacO9sy9C-y7jI2uavH"

#include <DHT.h>
#include <BlynkSimpleEsp32.h>
#include <ESP32Servo.h>
#include <WiFi.h>
#include <WiFiClient.h>

// Wi-Fi credentials
char ssid[] = "eloo";
char pass[] = "88888888";

// Sensor constants and variables
const int AirValue = 2540;
const int WaterValue = 1350;
int soilMoistureValue = 0;
int soilmoisturepercent = 0;
int lastSoilMoisturePercent = -1; // Untuk membandingkan nilai sebelumnya

// DHT22 sensor
#define DATA_PIN 13
#define DHTTYPE DHT22
DHT sensor(DATA_PIN, DHTTYPE);
float lastTemp = -999, lastHumidity = -999; // Nilai awal untuk perbandingan

// Ultrasonic sensor (updated)
#define SR04_TRIGGER 33      // Pin untuk Trigger Ultrasonic
#define SR04_ECHO 32         // Pin untuk Echo Ultrasonic
#define SOUND_SPEED 0.034    // Kecepatan suara dalam cm/µs
long duration;               // Durasi waktu yang diperlukan gelombang suara untuk kembali
float distanceCm;            // Jarak dalam sentimeter
float waterHeightCm;         // Ketinggian air dalam sentimeter
float lastWaterHeightCm = -1; // Untuk membandingkan nilai sebelumnya

// Servo motor
Servo myservo;
int posisiSekarang;
int lastServoPos = -1; // Untuk membandingkan nilai sebelumnya
int button = 2;
int manual = 3;

// Threshold kelembaban tanah untuk membuka dan menutup pintu air
#define SOIL_MOISTURE_THRESHOLD_OPEN 30  // Buka pintu air saat kelembaban < 30%
#define SOIL_MOISTURE_THRESHOLD_CLOSE 60 // Tutup pintu air saat kelembaban > 60%

BlynkTimer timer;

// Fungsi untuk membaca dan mengirim data soil moisture ke Blynk
void readSoilMoisture() {
  soilMoistureValue = analogRead(35);
  soilmoisturepercent = map(soilMoistureValue, AirValue, WaterValue, 0, 100);
  soilmoisturepercent = constrain(soilmoisturepercent, 0, 100);

  // Debug
  Serial.print("Soil Moisture: ");
  Serial.println(soilmoisturepercent);

  // Hanya kirim data jika ada perubahan
  if (soilmoisturepercent != lastSoilMoisturePercent) {
    Blynk.virtualWrite(V1, soilmoisturepercent);
    lastSoilMoisturePercent = soilmoisturepercent;
  }

  // Kontrol servo berdasarkan kelembaban tanah
  if (soilmoisturepercent < SOIL_MOISTURE_THRESHOLD_OPEN) {
    // Jika kelembaban tanah rendah, buka pintu air
    if (posisiSekarang != 90) {
      myservo.write(90);  // Buka pintu air
      Serial.println("Pintu air dibuka");
    }
  } else if (soilmoisturepercent > SOIL_MOISTURE_THRESHOLD_CLOSE) {
    // Jika kelembaban tanah tinggi, tutup pintu air
    if (posisiSekarang != 0) {
      myservo.write(0);   // Tutup pintu air
      Serial.println("Pintu air ditutup");
    }
  }
}

// Fungsi untuk membaca dan mengirim data suhu dan kelembaban dari DHT22 ke Blynk
void readDHT22() {
  float temp = sensor.readTemperature();
  float humidity = sensor.readHumidity();

  if (!isnan(temp) && !isnan(humidity)) {
    // Hanya kirim data jika ada perubahan signifikan
    if (abs(temp - lastTemp) > 0.5) {
      Blynk.virtualWrite(V2, temp);
      lastTemp = temp;
    }
    if (abs(humidity - lastHumidity) > 1) {
      Blynk.virtualWrite(V3, humidity);
      lastHumidity = humidity;
    }
  } else {
    Serial.println("Error reading DHT22 sensor data");
  }
}

// Fungsi untuk membaca dan mengirim data ketinggian air dari ultrasonic sensor ke Blynk
void readWaterHeight() {
  // Mengatur triggerPin ke LOW terlebih dahulu untuk memastikan tidak ada sinyal
  digitalWrite(SR04_TRIGGER, LOW);
  delayMicroseconds(2);

  // Mengirimkan pulse HIGH pada triggerPin selama 10 mikrodetik
  digitalWrite(SR04_TRIGGER, HIGH);
  delayMicroseconds(10);
  digitalWrite(SR04_TRIGGER, LOW);

  // Membaca durasi waktu pulse yang dipantulkan (echo)
  duration = pulseIn(SR04_ECHO, HIGH);

  // Menghitung jarak dalam sentimeter
  distanceCm = duration * SOUND_SPEED / 2; // Durasi dibagi 2 karena perjalanan bolak-balik

  // Menghitung ketinggian air berdasarkan jarak yang terukur
  waterHeightCm = 11.61 - distanceCm; // Misalnya, ketinggian air dihitung berdasarkan jarak sensor ke permukaan air

  // Hanya kirim data jika ada perubahan signifikan
  if (abs(waterHeightCm - lastWaterHeightCm) > 2) {
    Blynk.virtualWrite(V4, waterHeightCm);
    lastWaterHeightCm = waterHeightCm;
  }
}

// Fungsi untuk membaca status servo dan tombol, lalu mengirim data posisi servo ke Blynk
void readServo() {
  posisiSekarang = myservo.read();

  // Hanya kirim data jika ada perubahan posisi servo
  if (posisiSekarang != lastServoPos) {
    Blynk.virtualWrite(V5, posisiSekarang);
    lastServoPos = posisiSekarang;
  }
}

// Fungsi untuk mengontrol servo dari aplikasi Blynk
BLYNK_WRITE(V0) {
  int data = param.asInt();
  myservo.write(data);
  Blynk.virtualWrite(V0, data);
}

void setup() {
  Serial.begin(9600);
  sensor.begin();
  pinMode(SR04_TRIGGER, OUTPUT);
  pinMode(SR04_ECHO, INPUT);
  myservo.attach(15);
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  timer.setInterval(3000L, readSoilMoisture); // Setiap 3 detik
  timer.setInterval(5000L, readDHT22);       // Setiap 5 detik
  timer.setInterval(4000L, readWaterHeight); // Setiap 4 detik
  timer.setInterval(1000L, readServo);       // Setiap 1 detik
}

void loop() {
  Blynk.run();
  timer.run();
}
