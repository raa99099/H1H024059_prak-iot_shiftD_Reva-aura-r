1. Program yang Digunakan
Program disusun mengikuti struktur pada modul (pembacaan suhu dan kelembaban setiap 2 detik). Pada pelaksanaannya, sensor yang tersedia dan digunakan adalah DHT11 (bukan DHT22 seperti pada modul), sehingga baris DHTTYPE diubah menjadi DHT11; struktur logika program (dht.begin(), pembacaan, pengecekan isnan(), dan delay 2 detik) tetap mengikuti modul karena pustaka DHT.h mendukung kedua tipe sensor tersebut.
2. Konfigurasi Rangkaian
VCC sensor DHT ke 3.3V ESP32, pin DATA ke GPIO 4, dan GND sensor ke GND ESP32, sesuai Tabel 1.1 pada modul.







3. Hasil Pengamatan (cuplikan Serial Monitor)
No
Suhu (°C)
Kelembaban (%)
Status Pembacaan
1
28.90
56.00
Valid
2
28.90
56.00
Valid
3
28.90
55.00
Valid
4
28.90
55.00
Valid
5
28.90
54.00
Valid
6
28.00
59.00
Valid
7
28.00
57.00
Valid
8
28.00
56.00
Valid
9
28.50
53.00
Valid
10
27.60
60.00
Valid
11
27.10
59.00
Valid
12
27.10
60.00
Valid

 
Data lengkap hasil pembacaan (screenshot Serial Monitor) didokumentasikan secara terpisah dan dilampirkan pada tautan dokumentasi praktikum.
4. Analisis Kesesuaian dengan Spesifikasi
• Data suhu dan kelembaban berhasil dibaca dan ditampilkan pada Serial Monitor secara berkala, sesuai spesifikasi butir 1 dan 2 pada modul (data tampil setiap kurang lebih 2 detik dengan format “Suhu: .. °C, Kelembaban: .. %”).
• Sepanjang pengamatan tidak ditemukan hasil pembacaan bernilai NaN (Not a Number), yang berarti tidak terjadi kegagalan komunikasi antara ESP32 dan sensor selama pengujian; fitur pengecekan isnan() pada program belum sempat teruji untuk kondisi gagal baca, namun logikanya sudah benar sesuai spesifikasi butir 3.
• Tidak ditemukan error pada proses kompilasi maupun pembacaan sensor selama pengujian berlangsung (spesifikasi butir 4 terpenuhi).
• Nilai suhu terlihat relatif stabil (berkisar 27.10 °C – 28.90 °C) sedangkan kelembaban berubah lebih dinamis (53% – 60%), hal ini wajar mengingat kelembaban udara lebih sensitif terhadap kondisi lingkungan sekitar sensor dibandingkan suhu ruangan.
Kesimpulan: hasil Percobaan 1A telah sesuai dengan spesifikasi yang diharapkan pada modul.
