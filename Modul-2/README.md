
# Praktikum Internet of Things (IoT) - Modul 2

## Koneksi WiFi: Station (STA) dan Access Point (AP)

### Identitas

**Nama:** Reva Aura Ramadhani
**NIM:** H1H024059
**Mata Kuliah:** Praktikum Internet of Things
**Kode:** TK245005
**Modul:** 2
**Semester:** Genap 2026/5

---

## 1. Deskripsi

Praktikum Modul 2 membahas konfigurasi koneksi WiFi pada perangkat mikrokontroler menggunakan dua mode, yaitu **Station (STA)** dan **Access Point (AP)**.

Pada praktikum ini digunakan **NodeMCU ESP8266** sebagai pengganti ESP32 karena perangkat ESP32 tidak tersedia. Library yang digunakan juga disesuaikan menjadi `ESP8266WiFi.h`.

Praktikum terdiri dari:

1. Percobaan 2A - Station (STA)
2. Percobaan 2B - Access Point (AP)

---

# 2. Percobaan 2A - Station (STA)

## Tujuan

Percobaan ini bertujuan untuk menghubungkan NodeMCU ESP8266 ke jaringan WiFi sebagai **Station (STA)** serta menampilkan informasi koneksi melalui Serial Monitor.

Informasi yang ditampilkan meliputi:

* Status koneksi WiFi
* IP Address
* MAC Address
* RSSI
* Status LED

## Komponen

* NodeMCU ESP8266
* Kabel USB
* Jaringan WiFi/Hotspot
* Arduino IDE

## Library

```cpp
#include <ESP8266WiFi.h>
```

## Kode Program

```cpp
#include <ESP8266WiFi.h>

const char* ssid = "realme C53";
const char* password = "PASSWORD_WIFI";

const int ledPin = D4; // GPIO 2 sebagai LED indikator status koneksi

void setup() {
  Serial.begin(115200);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  // Set mode WiFi menjadi Station
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  // Jika berhasil terhubung
  Serial.println();
  Serial.println("WiFi berhasil terhubung!");

  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  Serial.print("MAC Address: ");
  Serial.println(WiFi.macAddress());

  Serial.print("RSSI (dBm): ");
  Serial.println(WiFi.RSSI());

  digitalWrite(ledPin, HIGH); // Nyalakan LED sebagai indikator
}

void loop() {
  // Cek status koneksi setiap 5 detik
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Status: Terhubung");
  } else {
    Serial.println("Status: Terputus");
    digitalWrite(ledPin, LOW);
  }

  delay(5000);
}
```

> **Catatan:** `PASSWORD_WIFI` digunakan sebagai pengganti password WiFi asli agar password tidak dipublikasikan di repository GitHub.

## Hasil Pengamatan

Setelah program dijalankan, NodeMCU berhasil terhubung ke jaringan WiFi. Serial Monitor menampilkan informasi koneksi berupa IP Address, MAC Address, RSSI, serta status koneksi.

LED pada pin D4 digunakan sebagai indikator ketika perangkat berhasil terhubung ke WiFi.

### Output Serial Monitor

```text
Menghubungkan ke WiFi....
WiFi berhasil terhubung!
IP Address: [IP Address]
MAC Address: [MAC Address]
RSSI (dBm): [RSSI]
Status: Terhubung
Status: Terhubung
Status: Terhubung
```

---

# 3. Percobaan 2B - Access Point (AP)

## Tujuan

Percobaan ini bertujuan untuk membuat NodeMCU ESP8266 menjadi **Access Point (AP)** sehingga perangkat lain dapat terhubung ke jaringan WiFi yang dibuat oleh NodeMCU.

Program juga digunakan untuk mengetahui jumlah perangkat yang sedang terhubung ke Access Point.

## Komponen

* NodeMCU ESP8266
* Kabel USB
* Laptop/PC
* Smartphone atau perangkat lain sebagai client
* Arduino IDE

## Library

```cpp
#include <ESP8266WiFi.h>
```

## Konfigurasi Access Point

| Parameter  | Nilai             |
| ---------- | ----------------- |
| SSID       | ESP32_AccessPoint |
| Password   | 12345678          |
| IP Address | 192.168.4.1       |
| Mode WiFi  | Access Point      |

## Kode Program

```cpp
#include <ESP8266WiFi.h>

const char* ap_ssid = "ESP32_AccessPoint";
const char* ap_password = "12345678";

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_AP);
  WiFi.softAP(ap_ssid, ap_password);

  IPAddress apIP = WiFi.softAPIP();

  Serial.println("Access Point aktif!");

  Serial.print("SSID : ");
  Serial.println(ap_ssid);

  Serial.print("IP Address : ");
  Serial.println(apIP);
}

void loop() {
  int jumlahClient = WiFi.softAPgetStationNum();

  Serial.print("Jumlah perangkat terhubung: ");
  Serial.println(jumlahClient);

  delay(5000);
}
```

## Hasil Konfigurasi

| No. | Parameter           | Nilai Konfigurasi | Hasil Pengamatan   |
| --: | ------------------- | ----------------- | ------------------ |
|   1 | SSID                | ESP32_AccessPoint | ESP32_AccessPoint  |
|   2 | Password            | 12345678          | Berhasil digunakan |
|   3 | IP Address          | 192.168.4.1       | 192.168.4.1        |
|   4 | Status AP           | Aktif             | Aktif              |
|   5 | SSID terdeteksi     | Ya                | Ya                 |
|   6 | Perangkat terhubung | -                 | Berhasil           |

## Pengamatan Jumlah Client

Jumlah perangkat yang terhubung diamati setiap 5 detik melalui Serial Monitor.

| No. | Waktu (s) | Jumlah Client | Keterangan            |
| --: | --------: | ------------: | --------------------- |
|   1 |         0 |             0 | Belum ada perangkat   |
|   2 |         5 |             0 | Belum ada perangkat   |
|   3 |        10 |             0 | Belum ada perangkat   |
|   4 |        15 |             0 | Belum ada perangkat   |
|   5 |        20 |             0 | Belum ada perangkat   |
|   6 |        25 |             0 | Belum ada perangkat   |
|   7 |        30 |             0 | Belum ada perangkat   |
|   8 |        35 |             1 | 1 perangkat terhubung |
|   9 |        40 |             1 | 1 perangkat terhubung |
|  10 |        45 |             2 | 2 perangkat terhubung |

### Output Serial Monitor

```text
Access Point aktif!
SSID : ESP32_AccessPoint
IP Address : 192.168.4.1

Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 0
Jumlah perangkat terhubung: 1
Jumlah perangkat terhubung: 1
Jumlah perangkat terhubung: 2
```

---

# 4. Perbandingan Mode STA dan AP

| Parameter                   | Station (STA)              | Access Point (AP)     |
| --------------------------- | -------------------------- | --------------------- |
| Fungsi                      | Terhubung ke jaringan WiFi | Membuat jaringan WiFi |
| NodeMCU berperan sebagai    | Client                     | Access Point          |
| Terhubung ke router/hotspot | Ya                         | Tidak                 |
| Membuat SSID sendiri        | Tidak                      | Ya                    |
| IP Address                  | Diperoleh dari jaringan    | 192.168.4.1           |
| Monitoring client           | Tidak                      | Ya                    |
| Library                     | ESP8266WiFi.h              | ESP8266WiFi.h         |

---

# 5. Analisis

Pada percobaan **Station (STA)**, NodeMCU ESP8266 berfungsi sebagai client yang terhubung ke jaringan WiFi yang tersedia. Program menggunakan `WiFi.mode(WIFI_STA)` untuk menentukan mode Station dan `WiFi.begin()` untuk memulai koneksi.

Setelah koneksi berhasil, program menampilkan IP Address, MAC Address, dan RSSI melalui Serial Monitor. LED pada pin D4 digunakan sebagai indikator bahwa koneksi WiFi berhasil.

Pada percobaan **Access Point (AP)**, NodeMCU ESP8266 berfungsi sebagai pembuat jaringan WiFi. Mode AP diatur menggunakan `WiFi.mode(WIFI_AP)`, kemudian jaringan dibuat menggunakan `WiFi.softAP()`.

Access Point menggunakan SSID `ESP32_AccessPoint` dan memiliki IP Address `192.168.4.1`. Jumlah perangkat yang terhubung dapat dipantau menggunakan `WiFi.softAPgetStationNum()`.

Berdasarkan hasil pengamatan, pada awal pengujian belum terdapat perangkat yang terhubung. Pada detik ke-35 terdapat 1 perangkat yang terhubung, kemudian pada detik ke-45 jumlah perangkat bertambah menjadi 2.

---

# 6. Kesimpulan

Berdasarkan praktikum yang telah dilakukan, NodeMCU ESP8266 berhasil digunakan untuk menerapkan koneksi WiFi dalam mode **Station (STA)** dan **Access Point (AP)**.

Pada mode STA, NodeMCU berhasil terhubung ke jaringan WiFi dan menampilkan informasi berupa IP Address, MAC Address, RSSI, serta status koneksi melalui Serial Monitor.

Pada mode AP, NodeMCU berhasil membuat jaringan WiFi sendiri dengan SSID `ESP32_AccessPoint` dan IP Address `192.168.4.1`. Program juga berhasil memantau jumlah perangkat yang terhubung secara berkala.

Dengan demikian, praktikum ini memberikan pemahaman mengenai penggunaan NodeMCU ESP8266 sebagai perangkat yang dapat terhubung ke jaringan WiFi maupun sebagai pembuat jaringan WiFi.

---

# 7. Struktur Repository

```text
Modul-2-IoT/
│
├── README.md
│
├── Percobaan_2A_STA/
│   └── STA.ino
│
└── Percobaan_2B_AP/
    └── AP.ino
```

---

## 8. Teknologi yang Digunakan

* **Board:** NodeMCU ESP8266
* **Bahasa Pemrograman:** C/C++ Arduino
* **IDE:** Arduino IDE
* **Library:** ESP8266WiFi
* **Koneksi:** WiFi
* **Mode:** Station (STA) dan Access Point (AP)

---

## 9. Dokumentasi

Dokumentasi yang dapat ditambahkan ke repository:

* Foto rangkaian NodeMCU
* Screenshot kode program
* Screenshot Serial Monitor
* Foto perangkat yang berhasil terhubung ke Access Point
* Hasil pengujian jumlah client

---

## 10. Identitas Praktikan

**Nama:** Reva Aura Ramadhani
**NIM:** H1H024059

**Praktikum Internet of Things - Modul 2**
**Universitas Jenderal Soedirman**
