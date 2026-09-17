
```markdown
# 📡 Modul 3: Protokol Komunikasi IoT (HTTP & MQTT)

[![Board](https://img.shields.io/badge/Board-NodeMCU%20ESP8266-blue.svg)](https://www.espressif.com/)
[![Protocol](https://img.shields.io/badge/Protocol-HTTP%20%7C%20MQTT-green.svg)](https://mqtt.org/)
[![Data Format](https://img.shields.io/badge/Format-JSON-yellow.svg)](https://www.json.org/)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen.svg)]()

Repositori ini berisi kode program, data hasil pengamatan, serta dokumentasi Buku Catatan Praktikum (BCP) dan Data Pengamatan (Dapeng) untuk **Modul 3 Praktikum Internet of Things (TK245005)**.

---

## 👤 Informasi Praktikan

* **Nama:** Reva Aura Ramadhani
* **NIM:** H1H024059
* **Shift:** D
* **Program Studi:** Teknik Komputer
* **Instansi:** Universitas Jenderal Soedirman
* **Asisten Praktikum:** Imedia Sholem Shoukat (H1D023088)

---

## 📁 Struktur Repositori

```text
.
├── HTTP_Post_ESP8266/
│   └── HTTP_Post_ESP8266.ino      # Program Percobaan 3A (HTTP POST)
├── MQTT_Publish_ESP8266/
│   └── MQTT_Publish_ESP8266.ino    # Program Percobaan 3B (MQTT Publish)
├── docs/
│   ├── BCP_Modul3_H1H024059.md     # Buku Catatan Praktikum Lengkap
│   └── Dapeng_Modul3_H1H024059.md  # Data Pengamatan
└── README.md                       # Dokumentasi Repositori

```

---

## 🚀 Ringkasan Percobaan

### Percobaan 3A: Komunikasi Data Menggunakan HTTP

Mengirimkan data sensor simulasi (`suhu`, `kelembaban`) serta data waktu operasi mikrokontroler (`millis()`) dalam format JSON ke server `https://httpbin.org/post` menggunakan metode **HTTP POST**.

* **Endpoint:** `https://httpbin.org/post`
* **Metode:** HTTP POST
* **Payload Format:** JSON
* **Header:** `Content-Type: application/json`
* **Response Status:** `200 OK`

### Percobaan 3B: Komunikasi Data Menggunakan MQTT

Mempublikasikan data sensor simulasi ke broker MQTT **HiveMQ** (`broker.hivemq.com:1883`) secara berkala menggunakan pola komunikasi **Publish-Subscribe**.

* **Broker:** `broker.hivemq.com`
* **Port:** `1883` (Non-TLS)
* **Topic:** `unsoed/H1H024057/farhan/sensor`
* **Payload Format:** JSON

---

## 💻 Cara Menggunakan / Menjalankan Program

1. Buka berkas `.ino` pada folder masing-masing percobaan menggunakan **Arduino IDE**.
2. Pastikan Board Manager **ESP8266** sudah terinstal.
3. Instal pustaka pendukung berikut melalui *Library Manager*:
* **ArduinoJson** (oleh Benoit Blanchon)
* **PubSubClient** (oleh Nick O'Leary)


4. Sesuaikan kredensial WiFi pada kode program:
```cpp
const char* ssid = "NAMA_WIFI_KAMU";
const char* password = "PASSWORD_WIFI_KAMU";

```


5. Pilih board **NodeMCU 1.0 (ESP-12E Module)** dan port COM yang sesuai.
6. Unggah program ke modul ESP8266, lalu buka **Serial Monitor** pada *baud rate* `115200`.

---

## 📊 Data Pengamatan Ringkas

### 1. HTTP POST Data Sample

```json
{
  "args": {},
  "data": "{\"suhu\":28.5,\"kelembaban\":65,\"waktu\":122878}",
  "files": {},
  "form": {},
  "headers": {
    "Content-Length": "44",
    "Content-Type": "application/json",
    "Host": "httpbin.org"
  },
  "json": {
    "kelembaban": 65,
    "suhu": 28.5,
    "waktu": 122878
  },
  "origin": "182.2.42.31",
  "url": "[https://httpbin.org/post](https://httpbin.org/post)"
}

```

### 2. MQTT Publish Sample

| Topic | Payload JSON | Status |
| --- | --- | --- |
| `unsoed/H1H024057/farhan/sensor` | `{"suhu":28.5,"kelembaban":65}` | Berhasil |

---

## ❓ Soal & Jawaban Modul

### A. Percobaan 3A (HTTP)

1. **Gambarkan diagram alur proses pengiriman data melalui HTTP POST pada program di atas!**
* **Jawab:** Mulai $\rightarrow$ Inisialisasi Serial (`115200`) $\rightarrow$ Hubungkan ESP8266 ke jaringan WiFi $\rightarrow$ Cek status koneksi WiFi $\rightarrow$ Jika belum terhubung, tunggu dan coba lagi $\rightarrow$ Jika terhubung, buat objek `WiFiClientSecure` (`setInsecure`) dan `HTTPClient` $\rightarrow$ Set header `Content-Type: application/json` $\rightarrow$ Buat dokumen JSON (`suhu`, `kelembaban`, `waktu`) $\rightarrow$ Ubah JSON menjadi string (`serializeJson`) $\rightarrow$ Kirim request menggunakan `http.POST()` $\rightarrow$ Evaluasi kode response HTTP $\rightarrow$ Tampilkan isi response server pada Serial Monitor $\rightarrow$ Tutup koneksi `http.end()` $\rightarrow$ Tunda 10 detik (`delay(10000)`) $\rightarrow$ Ulangi proses dari pengecekan WiFi.


2. **Apa fungsi dari perintah `http.addHeader("Content-Type", "application/json")`?**
* **Jawab:** Perintah tersebut memberi tahu server penerima bahwa format data yang dikirimkan dalam *request body* HTTP POST berbentuk dokumen JSON, sehingga server dapat memproses dan mengurai (*parse*) data tersebut dengan benar.


3. **Jelaskan arti kode response HTTP 200 dan sebutkan salah satu contoh kode response HTTP lain beserta artinya!**
* **Jawab:** Kode response HTTP **200 (OK)** berarti permintaan (*request*) dari client berhasil diterima, dipahami, dan diproses oleh server tanpa kendala. Contoh kode response lainnya adalah HTTP **404 (Not Found)**, yang menandakan bahwa URL/endpoint atau *resource* yang diminta tidak ditemukan di server.


4. **Modifikasi program agar ESP32/ESP8266 dapat mengirimkan data tambahan berupa waktu dalam milidetik sejak dinyalakan menggunakan `millis()`!**
* **Jawab:** Modifikasi dilakukan pada objek JSON sebelum fungsi `serializeJson()` dipanggil dengan menambahkan baris kode: `doc["waktu"] = millis();`.



---

### B. Percobaan 3B (MQTT)

1. **Apa fungsi dari topic pada protokol MQTT, dan mengapa topic yang digunakan perlu dibuat unik?**
* **Jawab:** *Topic* berfungsi sebagai alamat, kanal, atau kategori pengelompokan pesan untuk mengarahkan pengiriman data antara *publisher* dan *subscriber*. *Topic* perlu dibuat unik agar data yang dipublikasikan tidak saling menimpa atau tercampur dengan data milik perangkat lain yang terhubung ke broker publik yang sama.


2. **Jelaskan fungsi dari perintah `client.loop()` yang dipanggil pada setiap iterasi `loop()`!**
* **Jawab:** Perintah `client.loop()` berfungsi menjaga kesinambungan komunikasi dengan broker MQTT, memproses pesan masuk (*incoming messages*) jika ESP8266 bertindak sebagai subscriber, serta mengirimkan sinyal *keep-alive* (ping) agar broker tidak memutuskan koneksi client.


3. **Apa yang akan terjadi apabila koneksi ke broker MQTT terputus di tengah program berjalan?**
* **Jawab:** Jika koneksi terputus, pemanggilan `client.connected()` pada fungsi `loop()` akan bernilai `false`. Program secara otomatis akan memanggil fungsi `hubungkanMQTT()` untuk mencoba melakukan proses *reconnect* (menghubungkan kembali) ke broker secara berkala.



---

### C. Pertanyaan Analisis

1. **Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!**
* **Jawab:** Pada Percobaan 3A, ESP8266 berhasil mengirimkan data JSON (`suhu`, `kelembaban`, `waktu`) ke endpoint `httpbin.org/post` menggunakan HTTP POST dan menerima response `200 OK`. Pada Percobaan 3B, ESP8266 berhasil terhubung ke broker HiveMQ (`broker.hivemq.com:1883`) dan mempublikasikan data sensor secara berkala ke topic `unsoed/H1H024057/farhan/sensor`.


2. **Bandingkan besar overhead data dan pola komunikasi antara protokol HTTP dan MQTT berdasarkan hasil percobaan yang telah dilakukan!**
* **Jawab:** HTTP menggunakan pola *Request-Response* dengan *overhead* header data yang besar pada setiap pengiriman. MQTT menggunakan pola *Publish-Subscribe* melalui broker dengan *overhead* header yang sangat kecil (2 byte kontrol header) karena menggunakan koneksi TCP yang tetap terbuka (*persistent connection*).


3. **Untuk skenario pengiriman data sensor secara terus-menerus setiap beberapa detik dalam jangka waktu lama, protokol manakah (HTTP atau MQTT) yang lebih sesuai digunakan? Jelaskan alasannya!**
* **Jawab:** Protokol **MQTT** jauh lebih sesuai karena memanfaatkan *persistent connection* dan *overhead* data yang ringan, sehingga meminimalkan pemakaian *bandwidth* internet dan menghemat penggunaan daya listrik pada mikrokontroler.


4. **Bagaimana peran format JSON dalam mendukung interoperabilitas data antara perangkat IoT dan berbagai platform/aplikasi yang berbeda?**
* **Jawab:** JSON adalah format data berbasis teks yang bersifat *lightweight*, mudah dibaca, dan independen dari bahasa pemrograman. Karakteristik ini memungkinkan berbagai jenis perangkat keras (ESP8266), server web, *database*, hingga aplikasi *mobile/dashboard* untuk bertukar dan mengolah data tanpa kendala kompatibilitas.



---

## 📝 Lisensi & Catatan

Proyek ini dibuat dan diunggah sebagai bagian dari pemenuhan tugas mata kuliah Praktikum Internet of Things (TK245005) di Universitas Jenderal Soedirman.

```

```
