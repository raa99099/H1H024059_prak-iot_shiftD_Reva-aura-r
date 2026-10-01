# Modul 4 — Komunikasi Pertukaran Data Dua Arah (Publish dan Subscribe)

**Praktikum Internet of Things (TK245005)**  
**Program Studi Teknik Komputer — Universitas Jenderal Soedirman**

## Identitas Praktikan
* **Nama:** Reva Aura Ramadhani
* **NIM:** H1H024059
* **Shift / Kelompok:** D / Farhan
* **Nama Asisten:** Imedia Sholem Shoukat (H1D023088)

---

## Deskripsi Repository
Repository ini berisi program dan dokumentasi **Modul 4** mengenai implementasi komunikasi pertukaran data dua arah (*full duplex*) pada sistem IoT menggunakan protokol MQTT, pustaka `ArduinoJson` untuk deserialisasi data JSON, serta teknik *non-blocking* dengan `millis()`.

---

## 1. Percobaan 4A — Subscribe dan Deserialisasi Data JSON (Kendali PWM LED)

### Kode Program Modifikasi (`modul4A_PWM_LED.ino`)
```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// Konfigurasi WiFi
const char* ssid = "nneptuney";
const char* password = "9niraaeey9:3";

// Konfigurasi MQTT
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* topicPerintah = "unsoed/tk245004/farhan/perintah";

#define LED_PIN D2

WiFiClient espClient;
PubSubClient client(espClient);

void callback(char* topic, byte* payload, unsigned int length) {
  String pesan = "";
  for (unsigned int i = 0; i < length; i++) {
    pesan += (char)payload[i];
  }

  Serial.print("Pesan diterima [");
  Serial.print(topic);
  Serial.print("]: ");
  Serial.println(pesan);

  JsonDocument doc;
  DeserializationError error = deserializeJson(doc, pesan);

  if (error) {
    Serial.print("Gagal parsing JSON: ");
    Serial.println(error.c_str());
    return;
  }

  const char* perintah = doc["perintah"];
  
  // [MODIFIKASI] Membaca nilai "intensitas" dari JSON (Default 255 jika key tidak ada)
  int intensitas = doc["intensitas"] | 255; 

  if (String(perintah) == "ON") {
    // [MODIFIKASI] Mengatur tingkat kecerahan LED menggunakan sinyal PWM
    analogWrite(LED_PIN, intensitas); 
    Serial.print("Aktuator: ON | Intensitas PWM: ");
    Serial.println(intensitas);
  } 
  else if (String(perintah) == "OFF") {
    // [MODIFIKASI] Mematikan LED (Duty Cycle PWM = 0)
    analogWrite(LED_PIN, 0); 
    Serial.println("Aktuator: OFF | LED MATI");
  } 
  else {
    Serial.println("Perintah tidak dikenali");
  }
  Serial.println();
}

void hubungkanWiFi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi terhubung!");
}

void hubungkanMQTT() {
  while (!client.connected()) {
    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      client.subscribe(topicPerintah);
    } else {
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  analogWrite(LED_PIN, 0);
  
  hubungkanWiFi();
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    hubungkanMQTT();
  }
  client.loop();
}
```

### Penjelasan Kode Modifikasi Percobaan 4A
1. `int intensitas = doc["intensitas"] | 255;`: Membaca kunci `intensitas` dari payload JSON (misalnya `{"perintah": "ON", "intensitas": 150}`). Karakter `| 255` bertindak sebagai *fallback value* apabila *key* `intensitas` tidak disertakan dalam JSON.
2. `analogWrite(LED_PIN, intensitas);`: Menggantikan fungsi `digitalWrite()` untuk mengeluarkan sinyal PWM (*Pulse Width Modulation*) pada pin `D2` sehingga kecerahan LED bervariasi sesuai nilai dari JSON.

### Soal dan Jawaban Percobaan 4A
1. **Gambarkan diagram alur (flowchart) proses penerimaan dan pemrosesan pesan pada fungsi callback di atas!**
   * **Jawab:**  
     `Mulai Callback` -> `Konversi Payload Byte ke String (pesan)` -> `Deserialisasi JSON dengan deserializeJson(doc, pesan)` -> `Cek Error Parsing?`
     * Jika **Ya (Error)** -> Tampilkan "Gagal parsing JSON" ke Serial Monitor -> `Selesai Callback`.
     * Jika **Tidak (Sukses)** -> Ambil string `doc["perintah"]` -> `Cek Nilai Perintah`:
       * Jika `"ON"` -> `analogWrite(LED_PIN, intensitas)` -> Print status ke Serial Monitor.
       * Jika `"OFF"` -> `analogWrite(LED_PIN, 0)` -> Print status "LED MATI".
       * Lainnya -> Print "Perintah tidak dikenali".  
     -> `Selesai Callback`.

2. **Apa yang akan terjadi apabila pesan yang dipublikasikan bukan merupakan format JSON yang valid?**
   * **Jawab:** Fungsi `deserializeJson()` mengembalikan nilai error `DeserializationError`. Pengecekan `if (error)` terpenuhi, mencetak pesan kegagalan ke Serial Monitor, dan langsung melakukan `return` untuk menghentikan eksekusi `callback()`. Akibatnya, status kondisi pin LED tidak mengalami perubahan.

3. **Jelaskan mengapa fungsi `client.subscribe()` dipanggil di dalam fungsi `hubungkanMQTT()`, bukan di dalam `setup()`!**
   * **Jawab:** Koneksi MQTT dapat terputus sewaktu-waktu akibat kendala jaringan. Jika `client.subscribe()` diletakkan pada `setup()`, pendaftaran topic hanya dilakukan sekali saat perangkat booting. Memanggilnya di dalam `hubungkanMQTT()` memastikan ESP8266 otomatis mendaftar ulang (*re-subscribe*) ke topic perintah setiap kali koneksi ke broker MQTT berhasil dipulihkan.

4. **Modifikasi program agar data JSON yang diterima juga memuat nilai intensitas untuk mengatur kecerahan LED menggunakan PWM, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!**
   * **Jawab:** Kode program modifikasi beserta penjelasan per baris kodenya telah dicantumkan pada bagian atas bab ini.

---

## 2. Percobaan 4B — Pertukaran Data Dua Arah (Publish dan Subscribe Bersamaan)

### Kode Program Modifikasi (`modul4B_MultiTopic_Buzzer.ino`)
```cpp
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <DHT.h>

// Konfigurasi WiFi & MQTT
const char* ssid = "nneptuney";
const char* password = "9niraaeey9:3";
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;

// Topic MQTT
const char* topicData = "unsoed/tk245004/farhan/data";
const char* topicPerintah = "unsoed/tk245004/farhan/perintah";

// [MODIFIKASI] Topic Tambahan untuk Aktuator Kedua (Buzzer)
const char* topicBuzzer = "unsoed/tk245004/farhan/buzzer"; 

#define DHTPIN D4
#define DHTTYPE DHT11
#define LED_PIN D2

// [MODIFIKASI] Pin Hardware Buzzer
#define BUZZER_PIN D3 

DHT dht(DHTPIN, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

unsigned long waktuTerakhirPublish = 0;
const long intervalPublish = 5000;

void callback(char* topic, byte* payload, unsigned int length) {
  String pesan = "";
  for (unsigned int i = 0; i < length; i++) pesan += (char)payload[i];

  JsonDocument doc;
  if (deserializeJson(doc, pesan)) return;

  // [MODIFIKASI] Memeriksa asal topic yang menerima pesan
  if (String(topic) == topicPerintah) {
    const char* perintah = doc["perintah"];
    digitalWrite(LED_PIN, String(perintah) == "ON" ? HIGH : LOW);
    Serial.print("Kontrol LED -> ");
    Serial.println(perintah);
  } 
  else if (String(topic) == topicBuzzer) {
    const char* statusBuzzer = doc["buzzer"];
    digitalWrite(BUZZER_PIN, String(statusBuzzer) == "ON" ? HIGH : LOW);
    Serial.print("Kontrol Buzzer -> ");
    Serial.println(statusBuzzer);
  }
}

void hubungkanWiFi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);
  Serial.println("WiFi terhubung!");
}

void hubungkanMQTT() {
  while (!client.connected()) {
    String clientId = "ESP8266Client-" + String(random(0xffff), HEX);
    if (client.connect(clientId.c_str())) {
      // [MODIFIKASI] Subscribe ke dua topic sekaligus
      client.subscribe(topicPerintah);
      client.subscribe(topicBuzzer);
      Serial.println("Terhubung & Subscribe ke Topic LED dan Buzzer!");
    } else {
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(LED_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);
  dht.begin();

  hubungkanWiFi();
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) hubungkanMQTT();
  client.loop(); // Memproses incoming message secara berkala

  // Pengiriman data sensor suhu secara non-blocking (setiap 5 detik)
  if (millis() - waktuTerakhirPublish >= intervalPublish) {
    waktuTerakhirPublish = millis();
    float suhu = dht.readTemperature();

    if (!isnan(suhu)) {
      JsonDocument doc;
      doc["suhu"] = suhu;
      char buffer[128];
      serializeJson(doc, buffer);
      client.publish(topicData, buffer);
      Serial.print("Telemetry terkirim: ");
      Serial.println(buffer);
    }
  }
}
```

### Penjelasan Kode Modifikasi Percobaan 4B
1. `const char* topicBuzzer = "...";`: Mendefinisikan alamat topic MQTT baru khusus untuk mengontrol aktuator kedua (buzzer).
2. `client.subscribe(topicBuzzer);`: Mendaftarkan node ESP8266 pada broker HiveMQ agar mendengarkan pesan dari topic buzzer secara bersamaan dengan topic LED.
3. `if (String(topic) == topicBuzzer)`: Percabangan logika pada fungsi `callback()` untuk membedakan asal topic. Jika pesan berasal dari topic buzzer, program membaca kunci `doc["buzzer"]` untuk mengontrol pin `D3`.

### Soal dan Jawaban Percobaan 4B
1. **Mengapa penggunaan `delay()` yang lama sebaiknya dihindari pada program yang menggabungkan proses publish dan subscribe secara bersamaan?**
   * **Jawab:** Fungsi `delay()` bersifat *blocking*, yang berarti menghentikan seluruh eksekusi instruksi mikrokontroler. Jika `delay()` digunakan, panggilan fungsi `client.loop()` akan terhenti sehingga pesan *subscribe* yang dikirim pengguna tidak dapat langsung diproses (*lag/delay* respon). Jika delay terlalu lama, koneksi ke broker MQTT dapat terputus (*keep-alive timeout*).

2. **Jelaskan cara kerja mekanisme non-blocking menggunakan fungsi `millis()` pada program di atas!**
   * **Jawab:** Fungsi `millis()` mengembalikan durasi waktu dalam milidetik sejak board mulai dinyalakan. Mekanisme ini bekerja dengan membandingkan selisih waktu saat ini (`millis()`) dengan waktu terakhir publikasi dilakukan (`waktuTerakhirPublish`). Jika selisihnya belum mencapai `intervalPublish` (5000 ms), mikrokontroler melewati siklus publish data dan terus mengeksekusi instruksi berikutnya di `loop()` termasuk `client.loop()`.

3. **Apa yang akan terjadi apabila fungsi `client.loop()` jarang dipanggil (misalnya hanya sekali setiap 10 detik)?**
   * **Jawab:** Penerimaan pesan kendali MQTT (*subscribe*) menjadi sangat lambat dan tidak *real-time*. Perintah dari pengguna baru akan dieksekusi ESP8266 hingga 10 detik kemudian. Selain itu, buffer penerimaan data berisiko melimpah, dan broker dapat menganggap client tidak aktif lalu memutuskan koneksi.

4. **Modifikasi program agar menambahkan satu topic perintah baru untuk mengendalikan aktuator kedua (misalnya buzzer), dengan fungsi callback yang dapat membedakan topic mana yang menerima pesan, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!**
   * **Jawab:** Kode program modifikasi beserta penjelasannya telah dicantumkan pada bagian atas bab ini.

---

## 3. Pertanyaan Analisis Modul 4

1. **Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!**
   * **Jawab:** Pada Percobaan 4A, berhasil diimplementasikan fungsi *subscriber* MQTT dan deserialisasi JSON untuk mengendalikan aktuator LED secara *real-time* berdasarkan perintah dari MQTT Explorer. Pada Percobaan 4B, berhasil diintegrasikan mekanisme komunikasi dua arah (*full duplex*), yaitu mengirimkan data telemetry suhu sensor DHT11 ke topic data secara berkala menggunakan teknik *non-blocking* `millis()`, sekaligus menerima dan mengeksekusi perintah kontrol LED secara langsung tanpa terinterupsi.

2. **Bandingkan mekanisme komunikasi satu arah (publish saja, seperti pada Modul Praktikum 3) dengan komunikasi dua arah (publish dan subscribe) yang diimplementasikan pada modul ini!**
   * **Jawab:** Komunikasi satu arah (*publish* saja) hanya memungkinkan node IoT bertindak sebagai pengirim data pasif (*monitoring*), sehingga perangkat tidak dapat menerima instruksi balikan dari sistem luar. Komunikasi dua arah (*publish & subscribe*) memungkinkan pemantauan sekaligus pengendalian secara interaktif (*monitoring & controlling*), di mana node dapat melaporkan status fisiknya sekaligus merespons instruksi balik secara *real-time*.

3. **Mengapa pendekatan non-blocking (menggunakan `millis()`) lebih sesuai dibandingkan pendekatan blocking (menggunakan `delay()`) pada sistem IoT yang memerlukan komunikasi dua arah secara real-time?**
   * **Jawab:** Pendekatan *non-blocking* berbasis `millis()` menjaga *CPU cycle* terus berputar secara aktif tanpa interupsi, sehingga fungsi pendukung seperti `client.loop()` dapat dipanggil setiap milidetik. Hal ini memastikan pesan MQTT yang masuk pada fungsi *subscribe* langsung ditangani secepat mungkin (*real-time*). Sebaliknya, *blocking* berbasis `delay()` menghentikan eksekusi program, menyebabkan keterlambatan respon penerimaan instruksi kendali dan memicu risiko diskoneksi jaringan.

4. **Berikan contoh penerapan komunikasi dan pertukaran data dua arah pada sistem IoT nyata (sesuaikan dengan bidang peminatan masing-masing mahasiswa), dan jelaskan manfaatnya dibandingkan sistem yang hanya satu arah!**
   * **Jawab:** Pada bidang **Smart Agriculture (Pertanian Cerdas)**, sistem IoT dua arah digunakan untuk mengontrol kelembaban tanah. Perangkat mengirimkan data telemetry (*publish*) berupa tingkat kelembaban tanah dan suhu udara secara berkala ke server. Ketika nilai kelembaban berada di bawah ambang batas, server secara otomatis mempublikasikan perintah (*subscribe*) kepada perangkat untuk menyalakan pompa irigasi. Manfaatnya dibandingkan sistem satu arah adalah otomatisasi penuh; sistem tidak sekadar menyajikan data pemantauan, melainkan mampu melakukan tindakan koreksi fisik secara presisi dari jarak jauh.

---

## 4. Kesimpulan
1. Protokol MQTT mendukung komunikasi dua arah (*bidirectional*) yang efisien melalui pola *publish-subscribe* menggunakan *broker*.
2. Penerimaan data pada topik yang di-*subscribe* dikelola oleh fungsi *callback*, sementara data JSON mentah dikonversi menjadi tipe data usable dengan fungsi deserialisasi `deserializeJson()`.
3. Penerapan pemrograman *non-blocking* dengan fungsi `millis()` mutlak diperlukan untuk sistem IoT *full duplex* agar siklus `client.loop()` dapat memproses perintah masuk secara instan tanpa mengganggu jadwal publikasi data sensor.
