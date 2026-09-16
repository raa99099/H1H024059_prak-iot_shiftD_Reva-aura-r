# PRAKTIKUM IoT – AKUISISI SENSOR DHT22 DAN KENDALI AKTUATOR

## 1.5.4 Pertanyaan Praktikum – Akuisisi Data Sensor DHT22

### 1. Gambarkan diagram alur (flowchart) proses akuisisi data sensor DHT22!

Flowchart proses akuisisi data sensor DHT22:

```text
          MULAI
            |
            v
     Inisialisasi Serial
       dan Sensor DHT22
            |
            v
      Baca suhu dan
       kelembaban
            |
            v
     Apakah data sensor
          valid?
        /       \
      Tidak      Ya
       |          |
       v          v
 Tampilkan pesan   Tampilkan
 "Gagal membaca"   suhu & kelembaban
       |          |
       |          |
       +----+-----+
            |
            v
       Delay ± 2 detik
            |
            v
        Ulangi loop
```

### 2. Apa fungsi dari perintah `isnan()` pada program?

`isnan()` digunakan untuk memeriksa apakah hasil pembacaan sensor merupakan nilai **NaN (Not a Number)** atau bukan.

Pada program:

```cpp
if (isnan(suhu) || isnan(kelembaban)) {
  Serial.println("Gagal membaca data sensor!");
}
```

Jika pembacaan sensor gagal dan menghasilkan nilai NaN, program akan menampilkan pesan **"Gagal membaca data sensor!"**.

Jadi, fungsi `isnan()` adalah untuk **mengecek apakah data sensor valid atau terjadi kegagalan pembacaan**.

### 3. Mengapa diperlukan jeda minimal sekitar 2 detik antar pembacaan sensor DHT22?

Sensor DHT22 tidak dirancang untuk membaca data secara terus-menerus dalam waktu yang sangat cepat. Sensor membutuhkan waktu untuk melakukan pengukuran dan memperbarui data.

Jeda sekitar 2 detik diperlukan agar:

1. Sensor memiliki waktu untuk melakukan pengukuran.
2. Data yang dibaca lebih stabil.
3. Menghindari pembacaan sensor yang terlalu cepat.
4. Mengurangi kemungkinan pembacaan data gagal.

Jika sensor dibaca terlalu cepat, data dapat menjadi tidak akurat atau pembacaan dapat gagal.

---

## 1.5.4 – Modifikasi Program Rata-Rata 5 Kali Pembacaan

### Program

```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
}

void loop() {
  float totalSuhu = 0;
  float totalKelembaban = 0;
  int jumlahPembacaan = 0;

  for (int i = 0; i < 5; i++) {
    float suhu = dht.readTemperature();
    float kelembaban = dht.readHumidity();

    if (isnan(suhu) || isnan(kelembaban)) {
      Serial.println("Gagal membaca data sensor!");
    } else {
      totalSuhu += suhu;
      totalKelembaban += kelembaban;
      jumlahPembacaan++;
    }

    delay(2000);
  }

  if (jumlahPembacaan > 0) {
    float rataSuhu = totalSuhu / jumlahPembacaan;
    float rataKelembaban = totalKelembaban / jumlahPembacaan;

    Serial.print("Rata-rata Suhu: ");
    Serial.print(rataSuhu);
    Serial.println(" °C");

    Serial.print("Rata-rata Kelembaban: ");
    Serial.print(rataKelembaban);
    Serial.println(" %");
  } else {
    Serial.println("Tidak ada data sensor yang valid.");
  }

  delay(1000);
}
```

### Penjelasan baris kode yang ditambahkan

```cpp
float totalSuhu = 0;
```

Digunakan untuk menyimpan jumlah seluruh hasil pembacaan suhu.

```cpp
float totalKelembaban = 0;
```

Digunakan untuk menyimpan jumlah seluruh hasil pembacaan kelembaban.

```cpp
int jumlahPembacaan = 0;
```

Digunakan untuk menghitung jumlah pembacaan sensor yang berhasil dan valid.

```cpp
for (int i = 0; i < 5; i++) {
```

Digunakan untuk melakukan pembacaan sensor sebanyak **5 kali**.

```cpp
float suhu = dht.readTemperature();
float kelembaban = dht.readHumidity();
```

Digunakan untuk membaca nilai suhu dan kelembaban dari sensor DHT22.

```cpp
if (isnan(suhu) || isnan(kelembaban)) {
```

Memeriksa apakah pembacaan suhu atau kelembaban gagal.

```cpp
totalSuhu += suhu;
```

Menambahkan hasil pembacaan suhu ke total suhu.

```cpp
totalKelembaban += kelembaban;
```

Menambahkan hasil pembacaan kelembaban ke total kelembaban.

```cpp
jumlahPembacaan++;
```

Menambah jumlah pembacaan yang berhasil.

```cpp
float rataSuhu = totalSuhu / jumlahPembacaan;
```

Menghitung nilai rata-rata suhu.

```cpp
float rataKelembaban = totalKelembaban / jumlahPembacaan;
```

Menghitung nilai rata-rata kelembaban.

Dengan menggunakan rata-rata dari 5 pembacaan, hasil sensor dapat menjadi lebih stabil karena perubahan kecil atau noise pada satu pembacaan dapat dikurangi.

---

# 1.6 Percobaan 2A – Kendali Aktuator Relay Berdasarkan Data Sensor

## 1.6.4 Pertanyaan Praktikum

### 1. Mengapa diperlukan nilai ambang batas (threshold) dalam sistem kendali aktuator berbasis sensor?

Threshold digunakan sebagai **batas atau patokan untuk menentukan kapan aktuator harus menyala atau mati**.

Pada program:

```cpp
const float suhuThreshold = 30.0;
```

Artinya suhu **30°C** digunakan sebagai batas.

Jika:

```text
Suhu > 30°C
```

maka aktuator menyala.

Sedangkan jika:

```text
Suhu <= 30°C
```

maka aktuator mati.

Dengan adanya threshold, sistem dapat mengambil keputusan secara otomatis berdasarkan kondisi lingkungan.

---

### 2. Apa yang terjadi apabila `suhuThreshold` diturunkan menjadi 20.0?

Jika nilai threshold diturunkan menjadi:

```cpp
const float suhuThreshold = 20.0;
```

maka aktuator akan lebih mudah menyala.

Contohnya:

```text
Suhu = 25°C
Threshold = 20°C
```

Karena:

```text
25°C > 20°C
```

maka aktuator akan **ON**.

Jadi, semakin rendah nilai threshold, semakin sering aktuator dapat berada dalam kondisi menyala karena suhu lebih mudah melewati batas tersebut.

---

### 3. Apa perbedaan kendali kondisi tunggal dengan kendali menggunakan histerisis?

#### Kendali kondisi tunggal

Pada kendali kondisi tunggal hanya terdapat **satu threshold**.

Contoh:

```text
Suhu > 30°C → ON
Suhu ≤ 30°C → OFF
```

Jika suhu berada di sekitar 30°C dan terus berubah sedikit, aktuator dapat sering berganti kondisi ON dan OFF.

#### Kendali histerisis

Pada histerisis digunakan **dua threshold**, yaitu batas untuk menyalakan dan batas untuk mematikan aktuator.

Contoh:

```text
Suhu > 30°C → ON
Suhu < 28°C → OFF
Suhu 28–30°C → Pertahankan kondisi sebelumnya
```

Histerisis membuat aktuator tidak mudah berubah-ubah ketika suhu berada di sekitar batas.

---

# 1.6.4 – Modifikasi Program Menggunakan Histerisis

## Program

```cpp
#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22
#define RELAYPIN 26

DHT dht(DHTPIN, DHTTYPE);

const float suhuON = 30.0;
const float suhuOFF = 28.0;

bool statusAktuator = false;

void setup() {
  Serial.begin(115200);
  dht.begin();

  pinMode(RELAYPIN, OUTPUT);
  digitalWrite(RELAYPIN, LOW);
}

void loop() {
  float suhu = dht.readTemperature();

  if (isnan(suhu)) {
    Serial.println("Gagal membaca data sensor!");
  } else {

    if (suhu > suhuON) {
      statusAktuator = true;
    } 
    else if (suhu < suhuOFF) {
      statusAktuator = false;
    }

    digitalWrite(RELAYPIN, statusAktuator ? HIGH : LOW);

    Serial.print("Suhu: ");
    Serial.print(suhu);
    Serial.print(" °C -> ");

    if (statusAktuator) {
      Serial.println("Aktuator: ON");
    } else {
      Serial.println("Aktuator: OFF");
    }
  }

  delay(2000);
}
```

## Penjelasan kode

```cpp
const float suhuON = 30.0;
```

Menentukan batas suhu untuk menyalakan aktuator.

Jika suhu lebih dari 30°C, aktuator akan menyala.

```cpp
const float suhuOFF = 28.0;
```

Menentukan batas suhu untuk mematikan aktuator.

Jika suhu kurang dari 28°C, aktuator akan mati.

```cpp
bool statusAktuator = false;
```

Menyimpan kondisi aktuator.

`false` berarti aktuator mati dan `true` berarti aktuator menyala.

```cpp
if (suhu > suhuON) {
    statusAktuator = true;
}
```

Jika suhu lebih dari 30°C, status aktuator diubah menjadi ON.

```cpp
else if (suhu < suhuOFF) {
    statusAktuator = false;
}
```

Jika suhu kurang dari 28°C, status aktuator diubah menjadi OFF.

```cpp
digitalWrite(RELAYPIN, statusAktuator ? HIGH : LOW);
```

Mengirimkan kondisi status aktuator ke pin relay.

Jika `statusAktuator` bernilai `true`, pin menjadi HIGH. Jika `false`, pin menjadi LOW.

```cpp
if (statusAktuator) {
    Serial.println("Aktuator: ON");
} else {
    Serial.println("Aktuator: OFF");
}
```

Menampilkan status aktuator pada Serial Monitor.

### Cara kerja histerisis

| Kondisi Suhu | Status Aktuator                   |
| ------------ | --------------------------------- |
| > 30°C       | ON                                |
| 28°C – 30°C  | Mempertahankan kondisi sebelumnya |
| < 28°C       | OFF                               |

Contoh:

```text
Suhu naik menjadi 31°C → Aktuator ON
Suhu turun menjadi 29°C → Aktuator tetap ON
Suhu turun menjadi 27°C → Aktuator OFF
```

Dengan cara tersebut, aktuator tidak mudah berkedip atau berganti ON/OFF ketika suhu berada di sekitar 30°C.

---

# 1.7 Pertanyaan Analisis

## 1. Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!

### Percobaan 1A – Akuisisi Data DHT22

Pada percobaan pertama, sensor DHT22 digunakan untuk membaca data suhu dan kelembaban lingkungan. Data kemudian ditampilkan pada Serial Monitor. Program juga menggunakan `isnan()` untuk memastikan data yang diterima dari sensor valid. Hasil pembacaan dapat berubah sesuai kondisi lingkungan di sekitar sensor.

Pada modifikasi program, pembacaan dilakukan sebanyak 5 kali dan kemudian dihitung nilai rata-ratanya. Hasil rata-rata membuat data yang ditampilkan menjadi lebih stabil.

### Percobaan 2A – Kendali Aktuator

Pada percobaan kedua, data suhu dari DHT22 digunakan sebagai dasar untuk mengendalikan relay atau LED. Jika suhu melewati threshold, aktuator menyala. Jika suhu berada di bawah threshold, aktuator mati.

Pada modifikasi menggunakan histerisis, sistem menggunakan dua batas suhu. Aktuator menyala ketika suhu lebih dari 30°C dan mati ketika suhu kurang dari 28°C. Pada suhu antara 28°C sampai 30°C, aktuator mempertahankan kondisi sebelumnya.

Hasil tersebut menunjukkan bahwa sensor DHT22 dapat digunakan sebagai sumber data untuk mengendalikan aktuator secara otomatis.

---

## 2. Bagaimana pengaruh akurasi dan waktu tanggap (response time) sensor terhadap kecepatan reaksi aktuator pada sistem IoT?

Akurasi sensor berpengaruh terhadap ketepatan keputusan aktuator. Jika data sensor kurang akurat, sistem dapat mengambil keputusan yang kurang sesuai dengan kondisi sebenarnya.

Waktu tanggap sensor juga memengaruhi kecepatan aktuator. Jika sensor membutuhkan waktu lebih lama untuk menghasilkan data baru, aktuator juga akan membutuhkan waktu lebih lama untuk merespons perubahan kondisi.

Contohnya, ketika suhu meningkat melewati 30°C, sistem baru dapat menyalakan aktuator setelah sensor memberikan hasil pembacaan yang menunjukkan bahwa suhu sudah melewati threshold.

Jadi, **sensor yang akurat dan memiliki waktu respons yang sesuai akan membantu sistem IoT memberikan respons yang lebih tepat dan cepat.**

---

## 3. Bagaimana cara kerja sistem dalam mengubah data sensor menjadi keputusan kendali aktuator?

Proses kerja sistem dapat dijelaskan sebagai berikut:

```text
Sensor DHT22
     ↓
Akuisisi data suhu
     ↓
Pemeriksaan data
     ↓
Membandingkan suhu dengan threshold
     ↓
Pengambilan keputusan
     ↓
GPIO ESP32
     ↓
Relay / LED
     ↓
Aktuator ON atau OFF
```

Penjelasannya:

1. DHT22 membaca kondisi suhu lingkungan.
2. ESP32 menerima data dari sensor.
3. Program memeriksa apakah data valid.
4. Data suhu dibandingkan dengan nilai threshold.
5. ESP32 menentukan apakah aktuator harus ON atau OFF.
6. GPIO mengirimkan sinyal ke relay atau LED.
7. Aktuator bekerja sesuai keputusan sistem.

Jadi, sistem mengubah **data sensor menjadi keputusan kendali** melalui proses pembacaan, pengolahan, perbandingan, dan pemberian sinyal ke aktuator.

---

## 4. Bagaimana kombinasi akuisisi data sensor dan kendali aktuator dapat digunakan untuk membangun sistem IoT yang responsif?

Kombinasi sensor dan aktuator dapat digunakan untuk membuat sistem yang dapat **mendeteksi kondisi lingkungan dan memberikan tindakan secara otomatis**.

### Contoh pada Smart Farming

```text
Sensor suhu & kelembaban
          ↓
       ESP32
          ↓
   Analisis kondisi
          ↓
     Threshold
          ↓
   ┌──────┴──────┐
   ↓             ↓
Kondisi panas   Kondisi normal
   ↓             ↓
Aktuator ON    Aktuator OFF
   ↓
Kipas / pompa
```

Contohnya, sensor DHT22 membaca suhu dan kelembaban pada lahan atau greenhouse. Jika suhu terlalu tinggi, ESP32 dapat mengaktifkan kipas secara otomatis. Jika kondisi kembali normal, kipas dapat dimatikan.

### Contoh pada Smart Home

Sensor dapat digunakan untuk membaca suhu ruangan. Jika suhu melebihi batas tertentu, ESP32 dapat menyalakan kipas. Ketika suhu kembali turun, kipas dapat dimatikan.

Dengan demikian, kombinasi akuisisi sensor dan aktuator memungkinkan sistem IoT bekerja secara **otomatis, responsif, dan sesuai dengan kondisi lingkungan yang terdeteksi**.

---

# Kesimpulan

Pada praktikum ini telah dipelajari proses akuisisi data menggunakan sensor DHT22 dan pengendalian aktuator menggunakan ESP32. DHT22 digunakan untuk memperoleh data suhu dan kelembaban, kemudian data tersebut diproses oleh ESP32.

Penggunaan threshold memungkinkan sistem menentukan kapan aktuator menyala atau mati. Sementara itu, penggunaan histerisis dengan dua threshold dapat mengurangi perubahan status aktuator yang terlalu sering ketika nilai sensor berada di sekitar batas.

Secara keseluruhan, praktikum menunjukkan bahwa sensor, mikrokontroler, dan aktuator dapat bekerja bersama untuk membangun sistem IoT yang mampu memantau kondisi lingkungan dan memberikan respons secara otomatis.
