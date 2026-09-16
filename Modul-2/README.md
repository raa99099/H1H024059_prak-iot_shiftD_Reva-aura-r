# MODUL PRAKTIKUM 2

## KONFIGURASI JARINGAN ESP32

---

# 2.5.4 Pertanyaan Praktikum

### 1. Gambarkan diagram alur (flowchart) proses koneksi ESP32 ke jaringan WiFi pada program di atas!

**Jawaban:**

```text
              ┌─────────────┐
              │    MULAI    │
              └──────┬──────┘
                     │
                     ▼
          ┌─────────────────────┐
          │ Inisialisasi Serial │
          └──────────┬──────────┘
                     │
                     ▼
          ┌─────────────────────┐
          │ WiFi.mode(WIFI_STA) │
          └──────────┬──────────┘
                     │
                     ▼
          ┌─────────────────────┐
          │ WiFi.begin(SSID,    │
          │ PASSWORD)           │
          └──────────┬──────────┘
                     │
                     ▼
             ┌───────────────┐
             │ WiFi terhubung?│
             └───────┬───────┘
                 Tidak│      │Ya
                     │      │
                     ▼      ▼
             ┌───────────┐  ┌──────────────────┐
             │ Tunggu    │  │ Tampilkan pesan  │
             │ 500 ms    │  │ WiFi terhubung   │
             └─────┬─────┘  └────────┬─────────┘
                   │                 │
                   └──────►──────────┘
                                     │
                                     ▼
                            ┌─────────────────┐
                            │ Tampilkan IP    │
                            │ Address         │
                            └────────┬────────┘
                                     │
                                     ▼
                              ┌────────────┐
                              │   SELESAI  │
                              └────────────┘
```

---

### 2. Apa fungsi dari perintah `WiFi.mode(WIFI_STA)` pada program tersebut?

**Jawaban:**

Perintah `WiFi.mode(WIFI_STA)` berfungsi untuk mengatur ESP32 agar bekerja dalam **mode Station (STA)**. Dalam mode ini, ESP32 berperan sebagai klien yang terhubung ke jaringan WiFi yang sudah tersedia, seperti router atau hotspot smartphone.

---

### 3. Jelaskan apa yang terjadi apabila SSID atau password yang dimasukkan salah!

**Jawaban:**

Jika SSID atau password yang dimasukkan salah, ESP32 tidak dapat terhubung ke jaringan WiFi yang dituju. Status koneksi tidak akan mencapai `WL_CONNECTED`. Program akan terus memeriksa status koneksi apabila menggunakan perulangan `while`.

ESP32 juga tidak memperoleh IP address dari jaringan tersebut karena koneksi belum berhasil.

---

### 4. Modifikasi program agar ESP32 mencoba menghubungkan ulang (reconnect) secara otomatis apabila koneksi WiFi terputus!

**Jawaban:**

```cpp
#include <WiFi.h>

const char* ssid = "NAMA_WIFI";
const char* password = "PASSWORD_WIFI";

void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  Serial.print("Menghubungkan ke WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("WiFi terhubung!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

void loop() {

  if (WiFi.status() != WL_CONNECTED) {

    Serial.println("WiFi terputus! Mencoba reconnect...");

    WiFi.disconnect();
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
      delay(500);
      Serial.print(".");
    }

    Serial.println();
    Serial.println("WiFi berhasil terhubung kembali!");
    Serial.print("IP Address: ");
    Serial.println(WiFi.localIP());
  }

  delay(1000);
}
```

### Penjelasan kode tambahan

**`if (WiFi.status() != WL_CONNECTED)`**

Digunakan untuk memeriksa apakah ESP32 masih terhubung ke jaringan WiFi.

**`WiFi.disconnect();`**

Digunakan untuk memutus koneksi WiFi sebelumnya sebelum melakukan koneksi ulang.

**`WiFi.begin(ssid, password);`**

Digunakan untuk mencoba menghubungkan kembali ESP32 ke jaringan WiFi.

**`while (WiFi.status() != WL_CONNECTED)`**

Digunakan untuk terus memeriksa koneksi sampai ESP32 berhasil terhubung kembali.

**`delay(500);`**

Memberikan jeda 500 milidetik pada setiap pemeriksaan koneksi.

**`WiFi.localIP()`**

Digunakan untuk menampilkan IP address ESP32 setelah berhasil terhubung kembali. Fungsi-fungsi tersebut merupakan bagian dari pustaka WiFi.h yang digunakan pada konfigurasi WiFi ESP32.

---

# 2.7 Pertanyaan Praktikum

### 1. Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!

**Jawaban:**

Pada praktikum konfigurasi jaringan ESP32, percobaan dilakukan untuk memahami penggunaan beberapa mode jaringan WiFi. Pada mode **Station (STA)**, ESP32 berhasil dikonfigurasikan sebagai klien yang terhubung ke jaringan WiFi yang tersedia. Setelah berhasil terhubung, informasi jaringan seperti IP address dapat ditampilkan.

Pada mode **Access Point (AP)**, ESP32 dikonfigurasikan sebagai penyedia jaringan atau hotspot sehingga perangkat lain seperti laptop atau smartphone dapat terhubung langsung ke ESP32.

Pada mode **AP+STA**, ESP32 dapat berfungsi sebagai klien yang terhubung ke jaringan WiFi sekaligus menyediakan Access Point. Hasil percobaan menunjukkan bahwa ESP32 dapat menjalankan fungsi jaringan sesuai dengan mode yang dikonfigurasikan.

---

### 2. Bagaimana pengaruh kekuatan sinyal (RSSI) terhadap kestabilan koneksi WiFi pada perangkat IoT?

**Jawaban:**

RSSI (*Received Signal Strength Indicator*) menunjukkan kekuatan sinyal WiFi yang diterima oleh ESP32 dan dinyatakan dalam satuan dBm. Semakin kuat sinyal yang diterima, koneksi WiFi umumnya lebih stabil. Sebaliknya, sinyal yang lemah dapat menyebabkan koneksi menjadi kurang stabil, komunikasi data terganggu, atau koneksi terputus.

Oleh karena itu, kekuatan sinyal merupakan salah satu parameter yang perlu diperhatikan pada perangkat IoT yang menggunakan koneksi WiFi. Fungsi `WiFi.RSSI()` dapat digunakan untuk membaca kekuatan sinyal WiFi pada ESP32.

---

### 3. Bagaimana cara kerja ESP32 dalam membedakan peran sebagai klien (Station) dan sebagai penyedia jaringan (Access Point)?

**Jawaban:**

ESP32 membedakan perannya berdasarkan mode WiFi yang dikonfigurasikan.

Pada **mode Station (STA)**, ESP32 berperan sebagai **klien** yang terhubung ke jaringan WiFi yang sudah tersedia, seperti router atau hotspot smartphone.

Sedangkan pada **mode Access Point (AP)**, ESP32 berperan sebagai **penyedia jaringan** atau hotspot yang dapat diakses langsung oleh perangkat lain tanpa membutuhkan router eksternal.

Pemilihan mode tersebut dilakukan melalui konfigurasi WiFi pada program ESP32.

---

### 4. Bagaimana kombinasi mode Station dan Access Point (AP+STA) dapat dimanfaatkan dalam skenario nyata sistem IoT, misalnya pada proses konfigurasi awal perangkat (provisioning)?

**Jawaban:**

Mode **AP+STA** memungkinkan ESP32 terhubung ke jaringan WiFi yang sudah tersedia sebagai Station sekaligus menyediakan Access Point untuk perangkat lain.

Dalam proses **provisioning**, pengguna dapat terhubung ke Access Point yang dibuat ESP32 melalui smartphone atau laptop. Pengguna kemudian dapat memberikan informasi jaringan WiFi yang akan digunakan oleh ESP32. Setelah mendapatkan konfigurasi tersebut, ESP32 dapat terhubung ke jaringan WiFi sebagai Station.

Dengan cara ini, ESP32 dapat digunakan untuk proses konfigurasi awal perangkat IoT tanpa harus mengatur SSID dan password melalui perubahan program secara langsung. Mode AP+STA merupakan gabungan antara fungsi Station dan Access Point.

---

# 2.8 Mengakhiri Percobaan

Setelah praktikum selesai dilakukan, beberapa hal yang perlu diperhatikan adalah:

1. Memastikan seluruh perangkat dan rangkaian praktikum telah dimatikan dan dilepas dengan benar.
2. Memastikan meja praktikum dalam keadaan rapi dan bersih sebelum meninggalkan ruang praktikum.
