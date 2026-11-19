# 🌊 Penjelasan Mendetail Sistem OceanSmart

Dokumen ini menjelaskan secara komprehensif cara kerja **OceanSmart**, sebuah platform monitoring Kawasan Konservasi Laut yang menggabungkan IoT, Sistem Peringatan Dini, Digital Twin (3D), dan Kecerdasan Buatan (AI).

---

## 1. Mekanisme Data Real-Time (Siklus 10 Detik)

Salah satu inti dari sistem OceanSmart adalah aliran data waktu-nyata (real-time) yang bekerja tanpa henti di belakang layar (*background task*). Berikut adalah rincian alurnya:

### A. Backend Scheduler (Cron Job)
Sistem backend (menggunakan FastAPI Python) memiliki sebuah fungsi **Scheduler** yang berjalan berulang kali secara otomatis setiap **10 detik**. 
- Pada setiap siklus (interval 10 detik), backend akan memeriksa seluruh 28 sensor yang terdaftar di database.
- Backend memanggil fungsi `generate_reading()` untuk menciptakan data baru. Nilai ini disesuaikan dengan "Karakteristik Wilayah" (misalnya wilayah Raja Ampat memiliki probabilitas anomali sangat rendah, sedangkan perairan Cirebon lebih tinggi).

### B. Analisa Anomali Kualitas Air
Setiap data yang dihasilkan (Suhu, pH, Salinitas, DO, dan Kekeruhan) akan dilewatkan pada fungsi evaluasi *threshold* (ambang batas). Terdapat tiga tingkatan:
- **Normal**: Seluruh parameter aman.
- **Waspada**: Nilai mendekati batas ekstrem (misal Suhu: 29.5°C - 30.0°C).
- **Bahaya**: Nilai di luar ambang batas wajar (misal Suhu > 30.0°C), yang mengindikasikan adanya *Marine Heatwave* atau pencemaran.

### C. Server-Sent Events (SSE) Broadcast
Jika terdeteksi status **Waspada** atau **Bahaya**:
1. Backend langsung mencatat kejadian ini di tabel `Alert` di database (MySQL).
2. Backend akan melakukan *Broadcast* (penyiaran langsung) melalui koneksi **SSE (Server-Sent Events)** ke seluruh perangkat Frontend yang sedang aktif.
3. Dampaknya: Layar pengguna (Dashboard, Monitoring) akan memunculkan lencana (badge) peringatan baru, menampilkan warna merah, tanpa mengharuskan pengguna me-refresh halaman web (Polling Otomatis).

---

## 2. Digital Twin & Visualisasi 3D (Model `.glb`)

**Digital Twin** pada OceanSmart tidak sekadar menampilkan grafik angka, namun merupakan replika digital kawasan laut dalam bentuk tiga dimensi (3D) interaktif menggunakan model **GLB**.

### A. Format File GLB
File berekstensi `.glb` (GL Transmission Format Binary) digunakan sebagai standar format 3D karena ukurannya yang ringan dan mampu menyimpan data geometri, material, dan animasi (seperti gerakan berenang ikan) dalam satu file yang optimal untuk platform Web.

### B. Pemetaan Berdasarkan Zona Kedalaman
Sistem membagi penempatan model 3D berdasarkan parameter kedalaman air dari sensor:
- **0 - 5 meter** (Permukaan / Terumbu Karang Dangkal)
- **5 - 15 meter** (Menengah)
- **15 - 30 meter** (Dalam)

Setiap biota yang dimuat di database (seperti Penyu, Hiu, Terumbu Karang) memiliki properti `zona_kedalaman`. Frontend membaca konfigurasi ini dan memuat file `.glb` yang relevan ke dalam kanvas 3D (misalnya menggunakan pustaka *Three.js*).

### C. Lab Simulasi (Skenario "What-If")
Pengguna dapat menguji kondisi laut buatan.
- Jika pengguna memilih skenario **"Asidifikasi" (Pengasaman Laut)**, sistem di Frontend akan memodifikasi (intercept) data yang datang, memaksa pH turun ke angka ekstrem (misal pH < 7.0).
- Hal ini berdampak langsung pada indikator *Ocean Health Index (OHI)* dan memicu sistem peringatan berjalan seolah-olah terjadi bencana sungguhan. Nilai *baterai sensor* juga disimulasikan habis secara berkala (10% per siklus).

---

## 3. Integrasi Chatbot OceanBot (AI)

OceanSmart memanfaatkan Google Gemini 2.0 (dan Mistral sebagai cadangan) untuk mendampingi pengguna melalui fitur obrolan **OceanBot**.

- **Konteks Real-Time**: Tidak seperti ChatGPT biasa, OceanBot diberikan *prompt kontekstual*. Backend (Python) mengambil data sensor terakhir (hasil dari generate 10 detik sebelumnya), data Alert, dan informasi Biota, lalu disisipkan ke dalam perintah API Gemini.
- **Hasilnya**: Saat pengguna bertanya, *"Kenapa suhu air di Wakatobi hari ini tinggi?"*, AI mampu menjawab dengan akurat karena AI tersebut membaca data langsung dari database OceanSmart Anda.
- **Sistem Fallback**: Apabila koneksi internet terputus atau batas pemakaian Gemini API tercapai, bot secara pintar akan merespon menggunakan data lokal dari dalam database MySQL.

---

## 4. Struktur Database dan Hak Akses (RBAC)

Platform membedakan apa yang bisa dilihat oleh setiap orang:
- **User (Pengguna Umum)**: Hanya bisa melihat data agregat dan mengakses Lab Simulasi.
- **Operator Wilayah**: Dibatasi oleh kolom `wilayah` di tabel `User`. Seorang operator Pangandaran tidak dapat mengedit / me-resolve peringatan yang terjadi di wilayah Indramayu.
- **Admin**: Memiliki kendali penuh (*Super User*) untuk membuat sensor baru. Ketika admin menekan "Tambah Sensor", backend tidak sekadar menyimpan titik lokasi, melainkan memanggil fungsi untuk otomatis membuat **7 hari riwayat data simulasi** agar sensor baru tersebut tidak kosong saat pertama kali dilihat di grafik.

### Skema Tabel Penting (models.py):
1. `Sensor`: Menyimpan lokasi (Lat/Lng) dan zona.
2. `SensorReading`: Tabel raksasa tempat data polling 10 detik direkam, berisi historis pH, Salinitas, Suhu, dsb.
3. `Alert`: Rekaman peringatan anomali (Waspada/Bahaya).
4. `Biota`: Menyimpan tautan (`foto_url`) untuk file 3D `.glb` serta status konservasinya.

## 5. Teknologi, Bahasa Pemrograman, dan Framework

OceanSmart dibangun menggunakan arsitektur modern (Modern Web Stack) yang memisahkan bagian antarmuka pengguna (Frontend) dengan server pemrosesan (Backend).

### A. Bagian Frontend (Antarmuka Pengguna)
- **Bahasa Pemrograman**: JavaScript / ES6+
- **Framework Utama**: **React.js**
- **Build Tool**: **Vite** (Digunakan karena proses *build* dan *hot-reload* yang sangat cepat dibanding Webpack tradisional).
- **Pemetaan (SIG/GIS)**: **Leaflet.js** (Digunakan untuk merender peta interaktif interaktif beserta *marker* lokasi sensor kawasan konservasi).
- **Grafik & Data Visualisasi**: Digunakan untuk membuat grafik tren (Suhu, pH, dsb) secara dinamis.
- **Visualisasi 3D (Digital Twin)**: Menggunakan pustaka pendukung seperti **Three.js** untuk merender objek `.glb` biota laut secara interaktif langsung di peramban web (browser).

### B. Bagian Backend (Server dan API)
- **Bahasa Pemrograman**: **Python** (Versi 3.8+)
- **Framework Utama**: **FastAPI**
  FastAPI dipilih karena kemampuannya memproses permintaan HTTP dengan sangat cepat dan mendukung pemrograman asinkron (Asynchronous) secara bawaan. Hal ini sangat krusial mengingat sistem harus memproses pembaruan dari 28 sensor setiap 10 detik.
- **Mekanisme Real-time**: **Server-Sent Events (SSE)**.
  Berbeda dengan WebSockets yang berjalan dua arah secara terus-menerus, OceanSmart menggunakan SSE karena sistem peringatan lebih banyak bersifat satu arah (Server mengirimkan notifikasi ke Klien/Frontend).
- **AI / Chatbot**: **Google Gemini 2.0 Flash API** dan **Mistral API**.
  Python di backend bertugas merangkum konteks data dari MySQL lalu mengirimkannya ke layanan LLM (Large Language Model) pihak ketiga tersebut.

### C. Basis Data (Database)
- **Sistem Manajemen Basis Data**: **MySQL**
- **ORM (Object-Relational Mapping)**: **SQLAlchemy**
  Digunakan di backend Python untuk mempermudah operasi *CRUD* tanpa menulis perintah SQL secara manual. SQLAlchemy juga menghubungkan tabel-tabel kompleks seperti pengguna, sensor, riwayat pembacaan sensor (readings), alert, dan biota.

---
*OceanSmart - Sistem Monitoring Konservasi Laut Cerdas Generasi Berikutnya.*
