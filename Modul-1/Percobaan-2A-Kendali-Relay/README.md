1. Program yang Digunakan
Program mengikuti struktur pada modul: pembacaan suhu, kemudian pembandingan terhadap nilai ambang batas (suhuThreshold = 30.0 °C) untuk menentukan status aktuator (relay/LED).

2. Konfigurasi Rangkaian
Rangkaian Percobaan 1A tetap terpasang, ditambah IN relay/anoda LED ke GPIO 26 (melalui resistor 220 Ohm untuk LED), VCC relay ke 5V (VIN) ESP32, dan GND relay/katoda LED ke GND, sesuai Tabel 2.1 dan Gambar 1.1 pada modul.


3. Hasil Pengamatan (cuplikan Serial Monitor)
No
Suhu (°C)
Status Aktuator (Relay/LED)
1
27.10
OFF
2
27.10
OFF
3
27.10
OFF
4
27.60
OFF
5
28.50
OFF
6
29.40
OFF
7
30.80
ON
8
31.80
ON
9
32.30
ON
10
33.30
ON
11
33.80
ON
12
34.20
ON
13
34.70
ON
14
35.60
ON

 
4. Analisis Kesesuaian dengan Spesifikasi
• Aktuator (disimulasikan dengan LED) tetap berstatus OFF selama suhu terbaca di bawah ambang batas 30.0 °C (27.10 °C – 29.40 °C), dan berubah menjadi ON tepat setelah suhu melewati 30.0 °C, dimulai pada pembacaan 30.80 °C hingga 35.60 °C. Hal ini sesuai dengan spesifikasi butir 1 dan 2 pada modul.
• Serial Monitor menampilkan data suhu dan status aktuator secara bersamaan dalam satu baris (format “Suhu: .. °C -> Aktuator: ..”), sesuai spesifikasi butir 3.
• Transisi status OFF ke ON terjadi tepat satu kali dan konsisten mengikuti kenaikan suhu (tidak terjadi perubahan status yang berosilasi/flip-flop), sehingga pola menyala-mati aktuator berjalan konsisten sesuai kondisi suhu tanpa error, memenuhi spesifikasi butir 4.
• Pengujian dilakukan dengan mendekatkan sumber panas ke sensor DHT sehingga suhu naik secara bertahap dari sekitar 27 °C hingga di atas 35 °C, dan aktuator merespons sesuai threshold yang ditentukan.
Kesimpulan: hasil Percobaan 2A telah sesuai dengan spesifikasi yang diharapkan pada modul.
