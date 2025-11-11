# 3223600029_Syauqi_Nabil_Falah_TugasRTOS2

Sebelum mulai memprogram, rangkailah ESP32 berikut ini dengan konfigurasi pin yang sesuai.

<img width="633" height="708" alt="image" src="https://github.com/user-attachments/assets/c4d0d1d4-1219-46ac-af00-3a31464146ec" />

Masukkan program berikut pada Wokwi:
# Program memilih integrasi semua Task pada ESP32-S3
```cpp
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <ESP32Servo.h>
#include <AccelStepper.h>

// =================== PIN DEFINITIONS ====================
#define LED_PIN        2
#define BUZZER_PIN     46
#define BUTTON_A       35
#define BUTTON_B       19
#define POT_INPUT      5
#define ENC_CLK        41
#define ENC_DT         40
#define OLED_SDA       38
#define OLED_SCL       39
#define SERVO_PIN      18
#define STP1           4
#define STP2           7
#define STP3           8
#define STP4           9

// =================== DEVICES ====================
Adafruit_SSD1306 screen(128, 64, &Wire, -1);
Servo myservo;
AccelStepper motor(AccelStepper::FULL4WIRE, STP1, STP3, STP2, STP4);

volatile int encValue = 0;
int prevClk = 0;

// =================== TASK FUNCTIONS ====================
void taskLED(void *pv) {
  pinMode(LED_PIN, OUTPUT);
  while (1) {
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    Serial.printf("[LED] Core %d\n", xPortGetCoreID());
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}

void taskBuzzer(void *pv) {
  pinMode(BUZZER_PIN, OUTPUT);
  while (1) {
    digitalWrite(BUZZER_PIN, HIGH);
    vTaskDelay(200 / portTICK_PERIOD_MS);
    digitalWrite(BUZZER_PIN, LOW);
    vTaskDelay(200 / portTICK_PERIOD_MS);
    Serial.printf("[BUZZER] Core %d\n", xPortGetCoreID());
  }
}

void taskButton(void *pv) {
  pinMode(BUTTON_A, INPUT_PULLUP);
  pinMode(BUTTON_B, INPUT_PULLUP);
  while (1) {
    Serial.printf("[BUTTON] A=%d B=%d Core %d\n",
                  digitalRead(BUTTON_A),
                  digitalRead(BUTTON_B),
                  xPortGetCoreID());
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}

void taskPot(void *pv) {
  while (1) {
    int val = analogRead(POT_INPUT);
    Serial.printf("[POT] %d Core %d\n", val, xPortGetCoreID());
    vTaskDelay(200 / portTICK_PERIOD_MS);
  }
}

void taskOLED(void *pv) {
  Wire.begin(OLED_SDA, OLED_SCL);
  if (!screen.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("[OLED] Failed");
    vTaskDelete(NULL);
  }

  while (1) {
    screen.clearDisplay();
    screen.setTextSize(1);
    screen.setTextColor(SSD1306_WHITE);
    screen.setCursor(0, 0);
    screen.printf("OLED Running\nCore %d", xPortGetCoreID());
    screen.display();
    vTaskDelay(500 / portTICK_PERIOD_MS);
  }
}

void taskEncoder(void *pv) {
  pinMode(ENC_CLK, INPUT);
  pinMode(ENC_DT, INPUT);
  prevClk = digitalRead(ENC_CLK);

  while (1) {
    int clk = digitalRead(ENC_CLK);
    if (clk != prevClk) {
      if (digitalRead(ENC_DT) == clk) encValue++; else encValue--;
      Serial.printf("[ENC] %d Core %d\n", encValue, xPortGetCoreID());
    }
    prevClk = clk;
    vTaskDelay(5 / portTICK_PERIOD_MS);
  }
}

void taskServo(void *pv) {
  myservo.attach(SERVO_PIN);
  while (1) {
    for (int p = 20; p <= 160; p += 4) {
      myservo.write(p);
      vTaskDelay(15 / portTICK_PERIOD_MS);
    }
    for (int p = 160; p >= 20; p -= 4) {
      myservo.write(p);
      vTaskDelay(15 / portTICK_PERIOD_MS);
    }
    Serial.printf("[SERVO] Core %d\n", xPortGetCoreID());
  }
}

void taskStepper(void *pv) {
  motor.setMaxSpeed(500);
  motor.setAcceleration(200);

  while (1) {
    motor.moveTo(300);
    while (motor.distanceToGo()) motor.run();
    motor.moveTo(-150);
    while (motor.distanceToGo()) motor.run();
    Serial.printf("[STEPPER] Core %d\n", xPortGetCoreID());
  }
}

// =================== SETUP ====================
void setup() {
  Serial.begin(115200);
  delay(300);
  Serial.println("[SETUP] Starting all tasks...");

  // ================= CREATE ALL TASKS =================
  xTaskCreatePinnedToCore(taskLED,     "LED",     3000, NULL, 1, NULL, 0);
  xTaskCreatePinnedToCore(taskBuzzer,  "BUZZER",  3000, NULL, 1, NULL, 0);
  xTaskCreatePinnedToCore(taskButton,  "BUTTON",  3000, NULL, 1, NULL, 0);
  xTaskCreatePinnedToCore(taskPot,     "POT",     3000, NULL, 1, NULL, 0);
  xTaskCreatePinnedToCore(taskOLED,    "OLED",    6000, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskEncoder, "ENCODER", 3000, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskServo,   "SERVO",   5000, NULL, 1, NULL, 1);
  xTaskCreatePinnedToCore(taskStepper, "STEPPER", 7000, NULL, 1, NULL, 1);
}

void loop() {
  // Nothing — all logic handled by tasks
}

```
untuk mengganti task maka selectedTask dapat diganti menjadi salah satu task yang terdapat pada enum berikut ini, setelah pemilihan task dilakukan, pemilihan core juga dapat di inisialisasi.

**TASK_LED**
Task ini berfungsi untuk menyalakan dan mematikan LED secara bergantian dengan interval waktu tertentu. Delay selama 500 ms digunakan untuk mengatur kecepatan kedipan agar terlihat jelas di mata.

**TASK_BUZZER**
Bagian ini mengatur buzzer agar berbunyi dengan pola tertentu, yaitu hidup selama 200 ms dan mati selama 200 ms secara berulang.

**TASK_BUTTON**
Task ini bertugas membaca dua tombol input yang terhubung ke pin BUTTON_A dan BUTTON_B. Keduanya diatur dengan mode INPUT_PULLUP, sehingga logika yang dibaca akan aktif rendah (LOW saat ditekan). Nilai dari masing-masing tombol akan ditampilkan di Serial Monitor.

**TASK_POT**
Fungsi dari task ini adalah membaca nilai dari potensiometer melalui pin analog. Nilai tersebut kemudian dikirim ke Serial Monitor agar kita bisa melihat perubahan tegangan saat potensiometer diputar. Nilai ini biasanya digunakan untuk mengatur parameter seperti kecepatan, intensitas, atau posisi motor secara manual. Pembacaan dilakukan setiap 200 ms agar respon tetap halus tanpa terlalu membebani prosesor.

**TASK_OLED**
Task ini menangani tampilan pada layar OLED dengan menggunakan komunikasi I2C. Setiap setengah detik, tampilan diperbarui untuk menjaga agar layar tetap aktif. Task ini berguna untuk menampilkan status, data sensor, atau pesan singkat dari sistem secara real-time.

**TASK_ENCODER**
Bagian ini membaca sinyal dari rotary encoder yang terdiri dari dua pin, yaitu CLK dan DT. Setiap kali posisi encoder berubah, task akan mendeteksi arah putaran dengan membandingkan kedua sinyal tersebut. Nilai hasil perhitungan disimpan di variabel encValue dan dicetak ke Serial Monitor. Dengan cara ini, kita bisa mengetahui apakah encoder diputar ke kanan atau kiri.

**TASK_SERVO**
Task ini menggerakkan motor servo dengan pola maju-mundur dari sudut 20 derajat hingga 160 derajat, lalu kembali lagi ke posisi awal. Pergerakan dilakukan bertahap setiap beberapa derajat dengan jeda 15 ms agar gerakannya halus.

**TASK_STEPPER**
Task terakhir ini berfungsi untuk mengontrol motor stepper menggunakan library AccelStepper. Motor digerakkan ke posisi 300 langkah, lalu kembali ke -150 langkah secara berulang. Setiap pergerakan dilakukan dengan akselerasi dan kecepatan tertentu agar gerakannya tidak tiba-tiba.

Hasil:

![Demo](https://github.com/user-attachments/assets/f8f4934a-7fff-4170-8cbf-f89089b4b524?raw=true)
_Hasil seluruh IO/Peripheral_





