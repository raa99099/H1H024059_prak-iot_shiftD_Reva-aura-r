# Modul 5 — Pengolahan dan Analisis Data IoT (Filtering, Analisis, dan Visualisasi)

Praktikum Internet of Things (TK245005)
Program Studi Teknik Komputer — Universitas Jenderal Soedirman

## Identitas Praktikan

- Nama: Reva Aura Ramadhani
- NIM: H1H024059
- Shift / Kelompok: D / Farhan
- Nama Asisten: Imedia Sholem Shoukat (H1D023088)

## Deskripsi Repository

Repository ini berisi program dan dokumentasi Modul 5 mengenai pengolahan data pada sistem IoT, yaitu edge processing pada ESP8266 (filter moving average, nilai minimum/maksimum, dan simpangan baku) serta server processing pada komputer menggunakan Python (akuisisi data MQTT, penyimpanan CSV, analisis statistik, deteksi anomali dengan z-score, dan visualisasi).

---

## 1. Percobaan 5A — Pengolahan Data Sensor pada ESP8266 (Moving Average dan Statistik)

### Kode Program Modifikasi (`modul5_pengolahan_data.ino`)

```cpp
/*
 * MODUL 5 - PERCOBAAN 5A
 * Pengolahan Data Sensor (Moving Average dan Statistik)
 * Nama : Reva Aura Ramadhani
 * NIM  : H1H024059
 * Board: ESP8266 NodeMCU 1.0
 * Sensor: Dummy / Mock Function dari asprak
 */

#include <Arduino.h>
#include <math.h>

// Jumlah sampel moving average
const int N = 5;
float bufferSuhu[N] = {0};
int indeks = 0;
int jumlah = 0;

// Variabel suhu minimum dan maksimum
float suhuMin = 1000.0;
float suhuMax = -1000.0;

// Fungsi mock pengganti sensor DHT22
float bacaSuhuMock() {
  static unsigned long jumlahBaca = 0;
  jumlahBaca++;

  // Suhu dasar berubah secara bertahap
  float suhuDasar = 28.0 + 2.0 * sin(jumlahBaca * 0.08);

  // Noise kecil untuk simulasi fluktuasi sensor
  float noise = random(-5, 6) * 0.1;

  float suhu = suhuDasar + noise;

  // Lonjakan suhu setiap 25 pembacaan
  if (jumlahBaca % 25 == 0) {
    suhu += 8.0;
  }

  return suhu;
}

// Menghitung moving average
float hitungMovingAverage(float nilaiBaru) {
  bufferSuhu[indeks] = nilaiBaru;

  indeks = (indeks + 1) % N;

  if (jumlah < N) {
    jumlah++;
  }

  float total = 0;

  for (int i = 0; i < jumlah; i++) {
    total += bufferSuhu[i];
  }

  return total / jumlah;
}

// Menghitung simpangan baku dari buffer
float hitungSimpanganBaku() {
  if (jumlah == 0) return 0;

  float rata = 0;

  for (int i = 0; i < jumlah; i++) {
    rata += bufferSuhu[i];
  }

  rata /= jumlah;

  float totalKuadratSelisih = 0;

  for (int i = 0; i < jumlah; i++) {
    float selisih = bufferSuhu[i] - rata;
    totalKuadratSelisih += selisih * selisih;
  }

  return sqrt(totalKuadratSelisih / jumlah);
}

void setup() {
  Serial.begin(115200);
  randomSeed(micros());

  Serial.println("MODUL 5 - PERCOBAAN 5A");
  Serial.println("ESP8266 - MOCK SENSOR");
  Serial.println("------------------------------");
}

void loop() {
  // Membaca suhu dummy
  float suhu = bacaSuhuMock();

  // Menghitung moving average
  float suhuFilter = hitungMovingAverage(suhu);

  // Mencatat suhu minimum dan maksimum
  if (suhu < suhuMin) suhuMin = suhu;
  if (suhu > suhuMax) suhuMax = suhu;

  // Menghitung simpangan baku
  float stdSuhu = hitungSimpanganBaku();

  // Menampilkan hasil ke Serial Monitor
  Serial.print("Waktu: ");
  Serial.print(millis() / 1000);
  Serial.print(" s");

  Serial.print(" | Mentah: ");
  Serial.print(suhu, 2);

  Serial.print(" | Filter: ");
  Serial.print(suhuFilter, 2);

  Serial.print(" | Min: ");
  Serial.print(suhuMin, 2);

  Serial.print(" | Max: ");
  Serial.print(suhuMax, 2);

  Serial.print(" | Std: ");
  Serial.println(stdSuhu, 2);

  delay(2000);
}
```

### Penjelasan Kode Modifikasi Percobaan 5A

- `const int N = 5;`: Jumlah sampel moving average, yaitu 5 pembacaan terakhir.
- `float bufferSuhu[N] = {0};`: Buffer melingkar untuk menyimpan N pembacaan terakhir, semua elemen diisi 0 di awal.
- `int indeks = 0;` dan `int jumlah = 0;`: Posisi penulisan berikutnya pada buffer dan jumlah sampel yang sudah terisi (agar rata-rata awal tidak terpengaruh elemen kosong).
- `float suhuMin = 1000.0;` dan `float suhuMax = -1000.0;`: Nilai awal ekstrem agar pembacaan pertama langsung menggantikan nilai minimum dan maksimum.
- `float bacaSuhuMock() { ... }`: Fungsi pengganti sensor DHT22 dari asisten. Suhu dasar `28.0 + 2.0 * sin(jumlahBaca * 0.08)` berubah bertahap, `random(-5, 6) * 0.1` menambah noise ±0,5 °C, dan `suhu += 8.0` setiap pembacaan ke-25 menghasilkan lonjakan suhu.
- `bufferSuhu[indeks] = nilaiBaru;`: Menyimpan pembacaan terbaru ke buffer.
- `indeks = (indeks + 1) % N;`: Memajukan indeks dan kembali ke 0 setelah mencapai N, sehingga data terbaru menimpa data paling lama.
- `if (jumlah < N) jumlah++;`: Menambah jumlah sampel sampai buffer penuh.
- `return total / jumlah;`: Mengembalikan rata-rata isi buffer sebagai data hasil filter.
- `float hitungSimpanganBaku() { ... }`: **[MODIFIKASI]** Fungsi baru untuk menghitung simpangan baku isi buffer.
- `if (jumlah == 0) return 0;`: **[MODIFIKASI]** Mencegah pembagian dengan nol apabila belum ada data.
- `rata += bufferSuhu[i]; ... rata /= jumlah;`: **[MODIFIKASI]** Menghitung rata-rata isi buffer.
- `float selisih = bufferSuhu[i] - rata; totalKuadratSelisih += selisih * selisih;`: **[MODIFIKASI]** Menjumlahkan kuadrat selisih tiap data terhadap rata-rata.
- `return sqrt(totalKuadratSelisih / jumlah);`: **[MODIFIKASI]** Membagi dengan jumlah sampel lalu mengakarkannya sehingga diperoleh simpangan baku.
- `float stdSuhu = hitungSimpanganBaku();`: **[MODIFIKASI]** Memanggil fungsi simpangan baku pada setiap iterasi `loop()`.
- `Serial.print(" | Std: "); Serial.println(stdSuhu, 2);`: **[MODIFIKASI]** Menampilkan simpangan baku ke Serial Monitor dengan 2 angka desimal.
- `delay(2000);`: Pembacaan dilakukan setiap 2 detik.

### Data Pengamatan Percobaan 5A

Hasil keluaran Serial Monitor (baud rate 115200) pada ESP8266 NodeMCU 1.0:

| No. | Waktu (s) | Suhu Mentah (°C) | Moving Average / Filter (°C) | Suhu Min (°C) | Suhu Max (°C) | Std (°C) |
|---|---|---|---|---|---|---|
| 1 | 10 | 28.52 | 28.73 | 28.33 | 29.18 | 0.29 |
| 2 | 12 | 29.26 | 28.83 | 28.33 | 29.26 | 0.36 |
| 3 | 14 | 29.19 | 28.90 | 28.33 | 29.26 | 0.39 |
| 4 | 16 | 29.72 | 29.18 | 28.33 | 29.72 | 0.38 |
| 5 | 18 | 29.23 | 29.19 | 28.33 | 29.72 | 0.38 |
| 6 | 20 | 29.04 | 29.29 | 28.33 | 29.72 | 0.23 |
| 7 | 22 | 29.94 | 29.43 | 28.33 | 29.94 | 0.34 |
| 8 | 24 | 29.52 | 29.49 | 28.33 | 29.94 | 0.32 |
| 9 | 26 | 30.10 | 29.57 | 28.33 | 30.10 | 0.40 |
| 10 | 28 | 29.56 | 29.63 | 28.33 | 30.10 | 0.37 |
| 11 | 30 | 30.22 | 29.87 | 28.33 | 30.22 | 0.28 |
| 12 | 32 | 30.16 | 29.91 | 28.33 | 30.22 | 0.30 |
| 13 | 34 | 30.18 | 30.04 | 28.33 | 30.22 | 0.24 |
| 14 | 36 | 30.30 | 30.08 | 28.33 | 30.30 | 0.26 |

Keterangan: suhu mentah adalah keluaran fungsi mock sebelum difilter, Filter adalah moving average dari maksimal 5 pembacaan terakhir, Min dan Max adalah nilai terendah dan tertinggi sejak program berjalan, dan Std adalah simpangan baku isi buffer.

### Soal dan Jawaban Percobaan 5A

**1. Gambarkan diagram alur (flowchart) proses perhitungan moving average pada fungsi `hitungMovingAverage()`!**

Jawab:
Mulai → Terima `nilaiBaru` → Simpan ke `bufferSuhu[indeks]` → `indeks = (indeks + 1) % N` → Cek `jumlah < N`?
Jika Ya → `jumlah++`.
Jika Tidak → lanjut.
→ `total = 0` → Ulangi `i = 0` sampai `jumlah - 1`: `total += bufferSuhu[i]` → Hitung `total / jumlah` → Kembalikan hasil → Selesai.

**2. Jelaskan fungsi dari operasi `(indeks + 1) % N` pada program tersebut!**

Jawab: Operasi tersebut menggeser posisi penulisan satu langkah dan mengembalikannya ke 0 setelah mencapai N. Buffer bekerja melingkar (circular buffer): data terbaru menimpa data paling lama sehingga buffer selalu berisi N pembacaan terakhir tanpa indeks keluar dari batas array.

**3. Apa dampaknya terhadap data hasil filter apabila nilai N diperbesar menjadi 20 atau diperkecil menjadi 2?**

Jawab: Jika N = 20, data lebih halus dan lonjakan sesaat teredam, tetapi respons terhadap perubahan suhu yang sebenarnya menjadi lambat dan memori yang dipakai lebih besar. Jika N = 2, filter cepat merespons tetapi peredaman noise lemah sehingga data hampir sama dengan data mentah.

**4. Modifikasi program agar ESP32 juga menghitung simpangan baku dari data pada buffer, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!**

Jawab: Kode program modifikasi beserta penjelasan per baris kodenya telah dicantumkan pada bagian atas bab ini.

---

## 2. Percobaan 5B — Akuisisi, Analisis, dan Visualisasi Data

### Kode Program Modifikasi (`modul5_analisis_data.py`)

```python
import json
import time
import paho.mqtt.client as mqtt
import pandas as pd
import matplotlib.pyplot as plt

BROKER = "broker.hivemq.com"
PORT = 1883
TOPIC = "unsoed/tk245004/farhan/data"   # sama dengan topicData ESP
JUMLAH_SAMPEL = 30                       # jumlah data yang dikumpulkan

data = []

def on_connect(client, userdata, flags, reason_code, properties):
    print("Terhubung ke broker, kode:", reason_code)
    client.subscribe(TOPIC)

def on_message(client, userdata, msg):
    try:
        payload = json.loads(msg.payload.decode())
        suhu = float(payload["suhu"])
    except (ValueError, KeyError):
        print("Data tidak valid, diabaikan")
        return

    data.append({"waktu": time.strftime("%Y-%m-%d %H:%M:%S"), "suhu": suhu})
    print(f"[{len(data)}/{JUMLAH_SAMPEL}] suhu = {suhu}")

    if len(data) >= JUMLAH_SAMPEL:
        client.disconnect()

# --- Tahap 1: akuisisi data dari MQTT ---
client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)
client.on_connect = on_connect
client.on_message = on_message
client.connect(BROKER, PORT)
client.loop_forever()

# --- Tahap 2: simpan ke CSV ---
df = pd.DataFrame(data)
df.to_csv("data_suhu.csv", index=False)
print("Data tersimpan di data_suhu.csv")

# --- Tahap 3: analisis statistik dan deteksi anomali (z-score) data mentah ---
rata2 = df["suhu"].mean()
std = df["suhu"].std()

if std == 0:
    df["zscore"] = 0.0
else:
    df["zscore"] = (df["suhu"] - rata2) / std
df["anomali"] = df["zscore"].abs() > 2

print(f"Rata-rata : {rata2:.2f}")
print(f"Std deviasi: {std:.2f}")
print(f"Minimum : {df['suhu'].min():.2f}")
print(f"Maksimum : {df['suhu'].max():.2f}")
print(f"Jumlah anomali (data mentah): {df['anomali'].sum()}")

# --- [MODIFIKASI] Tahap 3b: moving average (jendela 5) sebelum z-score ---
df["suhu_ma"] = df["suhu"].rolling(window=5, min_periods=1).mean()

rata_ma = df["suhu_ma"].mean()
std_ma = df["suhu_ma"].std()

if std_ma == 0:
    df["zscore_ma"] = 0.0
else:
    df["zscore_ma"] = (df["suhu_ma"] - rata_ma) / std_ma
df["anomali_ma"] = df["zscore_ma"].abs() > 2

print(f"Jumlah anomali (data moving average): {df['anomali_ma'].sum()}")
df.to_csv("data_suhu_olah.csv", index=False)

# --- Tahap 4: visualisasi ---
plt.figure(figsize=(10, 4))
plt.plot(df.index, df["suhu"], marker="o", label="Suhu mentah")
plt.plot(df.index, df["suhu_ma"], color="green", label="Moving average (5)")  # [MODIFIKASI]
anom = df[df["anomali"]]
plt.scatter(anom.index, anom["suhu"], color="red", s=100, label="Anomali", zorder=3)
plt.axhline(rata2, color="gray", linestyle="--", label="Rata-rata")
plt.xlabel("Urutan data")
plt.ylabel("Suhu (°C)")
plt.title("Data Suhu dan Deteksi Anomali")
plt.legend()
plt.tight_layout()
plt.savefig("grafik_suhu.png")
plt.show()
```

### Penjelasan Kode Modifikasi Percobaan 5B

- `TOPIC = "unsoed/tk245004/farhan/data"`: Topic yang harus sama dengan `topicData` pada program ESP.
- `on_message()` dengan `try-except`: Memvalidasi payload JSON; data tidak valid diabaikan agar program tidak berhenti.
- `client.disconnect()`: Memutus koneksi setelah `JUMLAH_SAMPEL` (30) data terkumpul sehingga `loop_forever()` berhenti.
- `df.to_csv("data_suhu.csv", index=False)`: Menyimpan data beserta waktu penerimaan ke file CSV.
- `df["zscore"] = (df["suhu"] - rata2) / std`: Menghitung z-score tiap data; `df["anomali"] = ... > 2` menandai data dengan |z| > 2 sebagai anomali.
- `df["suhu_ma"] = df["suhu"].rolling(window=5, min_periods=1).mean()`: **[MODIFIKASI]** Menghitung moving average jendela 5 data pada kolom suhu. `min_periods=1` membuat 4 data pertama tetap dihitung dari data yang tersedia.
- `rata_ma = df["suhu_ma"].mean()` dan `std_ma = df["suhu_ma"].std()`: **[MODIFIKASI]** Menghitung rata-rata dan simpangan baku dari data yang sudah difilter.
- `df["zscore_ma"] = (df["suhu_ma"] - rata_ma) / std_ma`: **[MODIFIKASI]** Menghitung z-score berdasarkan data hasil moving average; jika `std_ma == 0` nilainya 0 untuk menghindari pembagian dengan nol.
- `df["anomali_ma"] = df["zscore_ma"].abs() > 2`: **[MODIFIKASI]** Menandai anomali pada data hasil filter dengan batas yang sama.
- `print(f"Jumlah anomali (data moving average): ...")`: **[MODIFIKASI]** Menampilkan jumlah anomali untuk dibandingkan dengan data mentah.
- `df.to_csv("data_suhu_olah.csv", index=False)`: **[MODIFIKASI]** Menyimpan hasil pengolahan (suhu_ma, z-score, anomali) ke CSV terpisah.
- `plt.plot(df.index, df["suhu_ma"], color="green", label="Moving average (5)")`: **[MODIFIKASI]** Menambahkan garis moving average pada grafik agar dapat dibandingkan dengan data mentah.

### Hasil Perbandingan

| Parameter | Data Mentah | Data Moving Average (5) |
|---|---|---|
| Rata-rata | 29,68 °C | 29,59 °C |
| Simpangan baku | 1,60 °C | 1,00 °C |
| Jumlah anomali (\|z\| > 2) | 1 (data ke-25, 37,42 °C) | 0 |

> Catatan: nilai di atas berasal dari simulasi data mock; ganti dengan hasil praktikum yang sebenarnya.

### Soal dan Jawaban Percobaan 5B

**1. Jelaskan fungsi dari blok try-except pada fungsi `on_message()` dan apa yang terjadi apabila blok tersebut dihilangkan!**

Jawab: Blok try-except menangkap error ketika payload bukan JSON yang valid, key `"suhu"` tidak ada, atau nilainya tidak dapat diubah menjadi angka. Data tidak valid diabaikan dan program tetap berjalan. Jika dihilangkan, satu pesan rusak akan menimbulkan exception sehingga program berhenti sebelum 30 data terkumpul.

**2. Jelaskan arti nilai z-score dan mengapa batas |z| > 2 digunakan sebagai penentu anomali!**

Jawab: Z-score menyatakan seberapa jauh suatu data dari rata-rata dalam satuan simpangan baku. Pada distribusi normal sekitar 95% data berada dalam ±2 simpangan baku, sehingga data di luar batas tersebut jarang terjadi dan layak dicurigai sebagai anomali.

**3. Apa kelemahan metode z-score apabila data memiliki tren naik atau turun yang tajam?**

Jawab: Rata-rata dan simpangan baku dihitung dari seluruh data sehingga tren tajam membuat data di ujung tampak jauh dari rata-rata dan salah dideteksi sebagai anomali, sedangkan lonjakan sebenarnya di tengah tren bisa lolos karena simpangan baku membesar. Anomali itu sendiri juga ikut menaikkan simpangan baku.

**4. Modifikasi program agar menggunakan metode moving average (jendela 5 data) pada data hasil CSV sebelum dihitung z-score-nya, kemudian bandingkan jumlah anomali yang terdeteksi, dan berikan penjelasan di setiap baris kode yang ditambahkan dalam bentuk README.md!**

Jawab: Kode program modifikasi beserta penjelasan per baris kodenya telah dicantumkan pada bagian atas bab ini. Data mentah menghasilkan 1 anomali, sedangkan data moving average menghasilkan 0 anomali karena lonjakan teredam oleh filter.

---

## 3. Pertanyaan Analisis Modul 5

**1. Uraikan hasil tugas pada praktikum yang telah dilakukan pada setiap percobaan!**

Jawab: Pada Percobaan 5A, ESP8266 membaca suhu mock, menghitung moving average 5 sampel, suhu minimum, maksimum, dan simpangan baku, lalu menampilkannya di Serial Monitor. Pada Percobaan 5B, program Python mengambil 30 data suhu melalui MQTT, menyimpannya ke CSV, menghitung statistik, mendeteksi anomali dengan z-score, dan membuat grafik.

**2. Bandingkan pengolahan data di sisi perangkat (Percobaan 5A) dengan pengolahan data di sisi komputer (Percobaan 5B) dari segi kecepatan, kompleksitas analisis, dan kebutuhan sumber daya!**

Jawab: Pengolahan di perangkat (edge) lebih cepat karena hasil langsung tersedia tanpa menunggu jaringan, tetapi analisisnya sederhana dan dibatasi memori serta CPU ESP8266. Pengolahan di komputer lebih lambat karena harus menunggu data terkumpul dan dikirim, tetapi mampu analisis kompleks (z-score, visualisasi, data historis) dengan sumber daya jauh lebih besar.

**3. Bagaimana pengaruh pemilihan ukuran jendela moving average terhadap kualitas data dan kecepatan respons sistem kendali apabila data tersebut digunakan untuk mengendalikan aktuator?**

Jawab: Jendela besar menghasilkan data lebih halus sehingga aktuator tidak mudah menyala-mati akibat noise, tetapi respons terhadap perubahan nyata menjadi lambat. Jendela kecil membuat respons cepat namun data masih berderau sehingga aktuator dapat bekerja berulang. Ukuran jendela dipilih sebagai kompromi sesuai kebutuhan sistem.

**4. Bagaimana hasil analisis dan deteksi anomali dapat dimanfaatkan untuk meningkatkan keandalan sistem IoT, misalnya pada sistem smart farming atau smart home?**

Jawab: Deteksi anomali menyaring pembacaan sensor yang salah sehingga aktuator tidak aktif karena data keliru, misalnya pompa penyiraman pada smart farming atau AC pada smart home. Anomali juga dapat menjadi tanda sensor rusak atau kondisi berbahaya (misalnya suhu naik mendadak) sehingga sistem dapat mengirim peringatan.

---

## 4. Kesimpulan

- Pengolahan data IoT dapat dilakukan di sisi perangkat (edge) maupun di sisi komputer sesuai kebutuhan kecepatan dan kompleksitas analisis.
- Filter moving average efektif meredam noise dan lonjakan sesaat, tetapi ukuran jendela memengaruhi kecepatan respons.
- Z-score berguna sebagai langkah awal deteksi anomali, namun kurang cocok untuk data dengan tren naik atau turun yang tajam.
- Kombinasi moving average dan z-score dapat mengurangi deteksi anomali palsu akibat noise.
