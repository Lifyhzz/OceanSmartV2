# Product Requirements Document (PRD)
# OceanSmart — Platform Monitoring Kawasan Konservasi Laut Berbasis IoT, AI, dan Digital Twin

**Versi:** 2.0 (Final — sesuai implementasi aktual)
**Tanggal:** Agustus 2026
**Status:** Aktif — Implementasi Selesai (Fase 1)
**Stack:** FastAPI + MySQL + React + Vite

---

## 1. Ringkasan Eksekutif

OceanSmart adalah platform digital berbasis **Sistem Informasi Geografis (SIG/GIS)** yang mengintegrasikan **monitoring kualitas air laut secara real-time** melalui sensor IoT, sistem **peringatan dini (early warning)**, **basis data biota laut berdasarkan kedalaman**, **Digital Twin 3D interaktif**, dan **chatbot AI berbasis Gemini/Mistral** untuk memudahkan akses informasi bagi pengelola kawasan konservasi, operator wilayah, dan masyarakat umum.

Seluruh data sensor menggunakan **data dummy yang dibangkitkan secara real-time** setiap 10 detik oleh backend scheduler, dengan karakteristik berbeda per wilayah, sebelum integrasi sensor IoT fisik di Fase 2.

---

## 2. Latar Belakang & Masalah

Kawasan konservasi laut di Indonesia menghadapi tantangan:
- Minimnya pemantauan kualitas air secara berkelanjutan dan real-time
- Data kondisi laut yang tersebar, tidak terintegrasi, dan sulit diakses publik
- Tidak ada sistem peringatan dini terhadap kondisi ekstrem (pemutihan karang, pencemaran, kenaikan suhu)
- Minimnya edukasi publik mengenai keanekaragaman hayati laut
- Pengelola kawasan kesulitan mengambil keputusan cepat karena data tidak tersaji secara visual dan terpusat

---

## 3. Tujuan Produk

1. Menyediakan peta interaktif kawasan konservasi laut berbasis SIG (Leaflet)
2. Memantau parameter kualitas air (pH, suhu, salinitas, DO, kekeruhan) secara real-time setiap 10 detik
3. Memberikan peringatan dini otomatis (Bahaya/Waspada) saat parameter melewati ambang batas
4. Menyajikan informasi biota laut berdasarkan zona kedalaman dengan model 3D interaktif
5. Memudahkan akses informasi melalui chatbot AI berbasis Gemini/Mistral dengan konteks data sensor
6. Mendukung pengambilan keputusan berbasis data melalui Digital Twin 3D lintas wilayah Indonesia

---

## 4. Target Pengguna & Role

| Role | Akses | Fungsi Utama |
|------|-------|-------------|
| **Admin / Developer** | Semua wilayah & data | CRUD Operator, CRUD Sensor, CRUD Biota, melihat seluruh data nasional |
| **Operator Wilayah** | Wilayah yang ditugaskan saja | Kelola sensor wilayah, selesaikan alert, tidak bisa akses Lab Simulasi |
| **Pengguna Umum** | Data publik | Dashboard, Monitoring, Biota, Peta, Lab Simulasi, Chatbot |

---

## 5. Fitur yang Sudah Diimplementasikan

### 5.1 Autentikasi & Manajemen Akun
- Login dengan email/password (case-insensitive)
- Login dengan Google OAuth
- Registrasi mandiri sebagai Pengguna Umum
- Halaman Profil: edit nama, nomor HP, ganti password
- Role-based access control (Admin, Operator, Pengguna)
- Admin buat akun Operator dari Panel Manajemen

**Akun Demo:**
- Admin: `admin@oceansmart.id` / `ocean123`
- Operator demo: `op.pangandaran@oceansmart.id`, `op.indramayu@oceansmart.id`, `op.bali@oceansmart.id`, `op.banyuwangi@oceansmart.id` / `op123`

### 5.2 Dashboard
- Ringkasan kondisi laut per wilayah/provinsi
- Ocean Health Index (OHI) berbasis rata-rata parameter sensor
- Grafik tren 24 jam per parameter (suhu, pH, DO, kekeruhan)
- Filter cascading: Provinsi → Kabupaten → Sensor
- Auto-refresh setiap 10 detik
- Mendukung 12 provinsi pesisir Indonesia

### 5.3 Peta SIG Interaktif
- Peta Leaflet dengan marker sensor per koordinat
- Overlay zona konservasi (Inti, Pemanfaatan Terbatas, Rehabilitasi, Pemanfaatan Umum)
- Klik marker → detail sensor
- Layer heatmap kesehatan

### 5.4 Monitoring Real-Time
- Grafik tren interaktif per parameter (24h, 7d, 30d, 90d)
- Nilai aktual + status aman/bahaya per parameter
- Coral Health Index card per sensor
- Auto-refresh polling 10 detik
- Filter per wilayah dan sensor

### 5.5 Database Biota Laut
- 17 spesies biota laut dengan model 3D GLB interaktif
- Filter berdasarkan zona kedalaman (0-5m, 5-15m, 15-30m, 5-30m)
- Pencarian berdasarkan nama umum/ilmiah
- Informasi: status konservasi IUCN, habitat, deskripsi
- Fallback data statis jika backend tidak aktif

### 5.6 Digital Twin 3D
- Cross-section lapisan kedalaman laut (Permukaan, Dangkal, Menengah, Dalam)
- Model 3D biota GLB per lapisan kedalaman dengan animasi swim
- Sensor dots per lapisan dengan Health Index real-time
- Simulasi What-If Multi-Parameter (suhu, pH, DO, kekeruhan) — efek lokal
- Filter wilayah/provinsi berdasarkan sensor dari DB (bukan dummy hardcoded)
- Telemetry mini-cards per sensor aktif (5 card parameter)

### 5.7 Peringatan Dini (Early Warning)
- Alert real-time via SSE (Server-Sent Events) dari backend
- Level: **Bahaya** (merah), **Waspada** (kuning), **Normal** (hijau)
- Setiap sensor menampilkan card status terkini
- Update setiap 10 detik sesuai generator realtime backend
- Tombol "Tandai Selesai" untuk operator/admin
- Filter: Prioritas, Wilayah, Sensor, Tanggal
- Toggle Grid/List view + Sort: Terbaru/Terlama/Tingkat Bahaya
- Badge sidebar hanya menampilkan alert bahaya/waspada aktif (tidak termasuk normal)
- Operator hanya melihat alert wilayahnya

### 5.8 Chatbot OceanBot (AI)
- Bertenaga Google Gemini 2.0 Flash (utama) + Mistral Small (fallback)
- Konteks real-time: data sensor + biota dari DB
- Kemampuan: analisis sensor by ID, penjelasan peringatan, info biota, edukasi kelautan
- Notifikasi alert muncul di chat setiap 60 detik saat ada bahaya aktif
- Animasi maskot: Wave (aman), Fall (ada alert), Yes (memproses)
- Notifikasi tidak menggulir otomatis saat user sedang baca jawaban chatbot
- Riwayat chat terhapus saat chatbot ditutup
- Jawaban tanpa simbol markdown (* atau **)

### 5.9 Panel Manajemen (Admin & Operator)
- Tab: Operator Wilayah (admin only), Stasiun Sensor IoT, Katalog Biota
- CRUD lengkap: Tambah/Edit/Hapus Operator, Sensor, Biota
- Saat sensor ditambah → otomatis generate 7 hari data historis dummy per karakteristik wilayah
- Operator hanya kelola sensor wilayahnya

### 5.10 Lab Simulasi IoT (Pengguna Umum Only)
- Skenario anomali: Marine Heatwave, Asidifikasi Air Laut, Badai & Limpasan, Normal
- Battery Drain: baterai sensor berkurang 10% per poll, sensor mati saat 0%
- Toggle online/offline/maintenance per sensor
- Isi ulang baterai sensor
- Reset simulator ke kondisi normal
- Efek simulasi mempengaruhi Dashboard, Monitoring, Alerts, Digital Twin via API interceptor
- SimulatorBanner muncul di semua halaman saat simulasi aktif
- Tidak tersedia untuk Admin dan Operator

---

## 6. Arsitektur Data Sensor

```
Backend Scheduler (10 detik)
        ↓
generate_reading() per sensor (karakteristik unik per wilayah)
        ↓
check_thresholds_and_create_alert() → level: bahaya/waspada/normal
        ↓
SensorReading + Alert di MySQL
        ↓
SSE Stream → AlertContext (frontend)
        ↓
Dashboard / Monitoring / Alerts / Chatbot / Sidebar badge
```

**28 sensor aktif tersebar di:**
Pangandaran (3), Sukabumi (2), Indramayu (2), Cirebon (2), Karawang (1), Subang (1), Parangtritis (2), Karimunjawa (3), Nusa Penida (3), Buleleng (2), Banyuwangi (3), Wakatobi (3), Bunaken (2), Komodo (2), Raja Ampat (3), Maluku Tengah (1)

---

## 7. Parameter Kualitas Air

| Parameter | Satuan | Batas Bahaya | Zona Waspada |
|-----------|--------|-------------|--------------|
| pH | - | < 7.5 atau > 8.5 | 7.5–7.7 atau 8.3–8.5 |
| Suhu | °C | < 26 atau > 30 | 26–26.5 atau 29.5–30 |
| Salinitas | ppt | < 30 atau > 35 | 30–30.5 atau 34.5–35 |
| DO (Oksigen) | mg/L | < 5 atau > 12 | 5–5.5 |
| Kekeruhan | NTU | > 10 | 7–10 |

---

## 8. Karakteristik Data per Wilayah

| Wilayah | Kondisi | Anomaly Rate |
|---------|---------|-------------|
| Raja Ampat | Terbaik, biodiversitas tinggi | 0.1% |
| Bunaken | Sangat bersih, wall dive | 0.2% |
| Wakatobi | Bersih, RAMSAR site | 0.3% |
| Nusa Penida | Bersih, habitat manta | 0.5% |
| Pangandaran | Baik, pesisir selatan | 1.0% |
| Karimunjawa | Baik, taman nasional | 1.0% |
| Banyuwangi | Sedang | 1.5% |
| Indramayu | Tercemar Pantura | 4.0% |
| Cirebon | Tercemar industri | 5.0% |

---

## 9. API Endpoint Utama

```
GET    /api/dashboard/summary
GET    /api/sensors                    (filter: X-User-Role, X-User-Wilayah, X-User-Provinsi)
GET    /api/sensors/{id}
POST   /api/sensors                    → auto-generate 7 hari data historis
PUT    /api/sensors/{id}/update
DELETE /api/sensors/{id}/delete
GET    /api/sensors/{id}/readings
GET    /api/alerts                     (filter: active_only, level, wilayah)
PATCH  /api/alerts/{id}/resolve
GET    /api/alerts/stream              (SSE real-time)
GET    /api/biota
POST   /api/biota
GET    /api/biota/{id}
PUT    /api/biota/{id}/update
DELETE /api/biota/{id}/delete
POST   /api/chatbot                    (Gemini → Mistral → offline fallback)
POST   /api/login
POST   /api/register
POST   /api/login/google
GET    /api/operators
POST   /api/operators
DELETE /api/operators/{id}
```

---

## 10. Konfigurasi Environment

**Backend (`backend/.env`):**
```
DATABASE_URL=mysql+pymysql://root:@localhost:3306/oceansmart
GEMINI_API_KEY=          # https://aistudio.google.com/app/apikey
MISTRAL_API_KEY=         # https://console.mistral.ai/
```

**Frontend (`frontend/.env`):**
```
VITE_GOOGLE_CLIENT_ID=   # https://console.cloud.google.com/
VITE_API_BASE_URL=http://localhost:8000/api
```

---

## 11. Setup & Instalasi

### Database
```sql
CREATE DATABASE oceansmart CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Backend
```bash
cd backend
python -m venv venv
venv\Scripts\activate        # Windows
pip install -r requirements.txt
copy .env.example .env       # isi DATABASE_URL dan API keys
uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```
Saat pertama jalan: auto-create tabel, seed 28 sensor, 4 operator demo, 17 biota, 30 hari data historis.

### Frontend
```bash
cd frontend
npm install
copy .env.example .env       # isi VITE_GOOGLE_CLIENT_ID
npm run dev
```
Akses: **http://localhost:5173**

---

## 12. Roadmap Fase Berikutnya

| Fase | Target |
|------|--------|
| **Fase 2** | Integrasi sensor IoT fisik (MQTT/LoRaWAN), ekspor laporan PDF/Excel |
| **Fase 3** | Aplikasi mobile native, notifikasi WhatsApp/email |
| **Fase 4** | Digital Twin 3D penuh (Three.js/CesiumJS), prediksi ML |

---

*Dokumen ini mencerminkan implementasi aktual OceanSmart versi Agustus 2026.*
