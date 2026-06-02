# Pemetaan Fitur, PBI, dan File Pendukung (WattCare)

Dokumen ini berisi daftar lengkap file source code yang bersangkutan dengan 4 fitur utama aplikasi **WattCare** (Fitur Pengingat, Fitur Berita, Fitur Chat AI, dan Fitur Profile/Admin). Format ini dirancang khusus untuk mempermudah koordinasi dan pembagian tugas dengan anggota tim pengembang lainnya.

---

## Ringkasan Cepat Pemetaan File

Berikut adalah tabel pemetaan ringkas antara fitur utama, PBI terkait, dan daftar berkas (*file paths*) yang menyusunnya di dalam proyek:

| Nama Fitur | ID PBI | Deskripsi PBI | File Pendukung Utama |
| :--- | :---: | :--- | :--- |
| **1. Fitur Pengingat** | **PBI-1.1** | Timer Klien & LocalStorage | `app/Http/Controllers/ReminderController.php`<br>`resources/views/reminder/index.blade.php`<br>`routes/web.php` |
| | **PBI-1.2** | Alarm & Push Notification | `resources/views/reminder/index.blade.php`<br>`public/sounds/myinstants.mp3` |
| | **PBI-1.3** | Pencatatan Log & Dampak Lingkungan | `app/Http/Controllers/ReminderController.php`<br>`app/Services/CalculatorService.php`<br>`app/Models/MonitoringLog.php`<br>`app/Models/Billing.php`<br>`app/Models/CO2Impact.php` |
| **2. Fitur Berita** | **PBI-2.1** | NewsAPI & Caching SDG 7 | `app/Services/NewsService.php`<br>`app/Http/Controllers/NewsController.php`<br>`routes/web.php` |
| | **PBI-2.2** | Mock News Fallback | `app/Services/NewsService.php`<br>`.env` |
| | **PBI-2.3** | Halaman Daftar & Detail Berita | `resources/views/news/index.blade.php`<br>`resources/views/news/show.blade.php`<br>`app/Http/Controllers/NewsController.php` |
| **3. Fitur Chat AI** | **PBI-3.1** | Ekstraksi Data Perangkat (Gemini API) | `app/Http/Controllers/ChatController.php`<br>`resources/views/chat/index.blade.php`<br>`routes/web.php` |
| | **PBI-3.2** | Validasi Kelengkapan Parameter | `app/Http/Controllers/ChatController.php` |
| | **PBI-3.3** | Kalkulasi Estimasi kWh Otomatis | `app/Http/Controllers/ChatController.php`<br>`resources/views/chat/index.blade.php` |
| **4. Fitur Profile/Admin** | **PBI-4.1** | Edit Profil & Validasi Daya VA | `app/Http/Controllers/ProfileController.php`<br>`app/Http/Requests/ProfileUpdateRequest.php`<br>`resources/views/profile/edit.blade.php` |
| | **PBI-4.2** | CRUD Manajemen User (Admin) | `app/Http/Controllers/Admin/UserController.php`<br>`app/Http/Middleware/AdminMiddleware.php`<br>`resources/views/admin/users/index.blade.php` |
| | **PBI-4.3** | CRUD Tarif Listrik per kWh (Admin) | `app/Http/Controllers/Admin/ElectricityRateController.php`<br>`app/Models/ElectricityRate.php`<br>`resources/views/admin/electricity_rates/index.blade.php` |

---

## Rincian File Berdasarkan Fitur

### 🔌 1. Fitur Pengingat (Reminder Feature)
Fitur ini berfokus pada alat bantu hitung mundur pemakaian peralatan listrik, memainkan audio alarm, dan mencatat penggunaannya langsung ke riwayat konsumsi listrik harian pengguna.

* **Controller (Logika Backend):**
  * [ReminderController.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/ReminderController.php)
    * `index()`: Mengirimkan daftar alat elektronik milik pengguna untuk pilihan opsi timer.
    * `store()`: Menerima masukan dari timer klien, menghitung konsumsi energi, dan menyimpan data log, billing, serta dampak lingkungan ke database.
* **View (Frontend & Interaksi Klien):**
  * [index.blade.php (Reminder)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/reminder/index.blade.php)
    * Menyediakan formulir input timer, sistem countdown berbasis JavaScript, penyimpanan keadaaan menggunakan `localStorage`, pemutaran alarm, modal pop-up, dan API notifikasi desktop.
* **Model (Struktur Data):**
  * [MonitoringLog.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Models/MonitoringLog.php): Menyimpan log durasi pemakaian alat dan total kWh.
  * [Billing.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Models/Billing.php): Menyimpan hitungan estimasi biaya berdasarkan tarif kWh saat pencatatan dilakukan.
  * [CO2Impact.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Models/CO2Impact.php): Menyimpan data emisi karbon CO2 yang dihasilkan dari pemakaian energi tersebut.
* **Services & Assets pendukung:**
  * [CalculatorService.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Services/CalculatorService.php): Melakukan kalkulasi matematika konversi Watt/Jam ke kWh, konversi kWh ke Rupiah, serta perhitungan emisi CO2.
  * `public/sounds/myinstants.mp3`: File suara alarm berformat MP3 yang dipicu ketika durasi habis.

---

### 📰 2. Fitur Berita (News Feature)
Fitur ini menampilkan agregasi berita dan tips efisiensi energi yang disaring secara otomatis dari rilis global NewsAPI, dengan cadangan data lokal berbahasa Indonesia jika API tidak aktif.

* **Controller (Logika Backend):**
  * [NewsController.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/NewsController.php)
    * `index()`: Mengarahkan data berita dari service ke tampilan utama.
    * `show($id)`: Menampilkan rincian berita penuh berdasarkan pencocokan ID unik artikel.
* **Services (Logika Integrasi API & Filter):**
  * [NewsService.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Services/NewsService.php)
    * Melakukan request HTTP ke `newsapi.org`, memilah kata kunci relevan SDG 7, mengeliminasi teks non-Latin, mengelola caching berita selama 1 jam, dan menyimpan daftar `getMockNews()` sebagai *fallback* data lokal.
* **View (Frontend):**
  * [index.blade.php (News)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/news/index.blade.php): Grid daftar artikel berita lengkap dengan pencarian dan deskripsi singkat.
  * [show.blade.php (News)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/news/show.blade.php): Layout terperinci untuk membaca satu artikel berita.

---

### 🤖 3. Fitur Chat AI (AI Chat Feature)
Asisten cerdas berbasis NLP yang mengekstrak informasi konsumsi alat dari bahasa sehari-hari pengguna menggunakan API Gemini.

* **Controller (Logika Utama & NLP Integrasi):**
  * [ChatController.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/ChatController.php)
    * `index()`: Menampilkan antarmuka UI percakapan obrolan.
    * `send()`: Mengirim teks input ke Gemini API, mengekstrak JSON dari teks AI, mengecek parameter yang kurang (*missing params validation*), dan menghitung estimasi kwh perangkat jika data sudah lengkap.
* **View (Frontend & Komponen Obrolan):**
  * [index.blade.php (Chat)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/chat/index.blade.php)
    * Halaman obrolan dengan gelembung chat visual, input teks AJAX, animasi mengetik (*typing indicator*), dan penanganan respons JSON dari server untuk merender teks tebal.

---

### 👤 4. Fitur Profile & Admin (Profile/Admin Feature)
Mengelola profil pengguna (termasuk validasi daya VA) serta kontrol admin menyeluruh untuk pengelolaan user dan tarif listrik per golongan VA.

* **Fitur Profil Pengguna:**
  * [ProfileController.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/ProfileController.php): Menangani render form profil dan pembaruan password terenkripsi.
  * [ProfileUpdateRequest.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Requests/ProfileUpdateRequest.php): Validasi formulir profil, mewajibkan email unik dan input kapasitas listrik `daya_va` bertipe integer dengan nilai minimal 450 VA.
  * [edit.blade.php (Profile)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/profile/edit.blade.php): Halaman form isian profil pengguna.
* **Fitur Manajemen User (Admin):**
  * [UserController.php (Admin)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/Admin/UserController.php): Resource controller (CRUD) untuk pengelolaan akun pengguna oleh administrator.
  * [AdminMiddleware.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Middleware/AdminMiddleware.php): Middleware keamanan untuk memastikan hanya pengguna bertipe peranan `admin` yang dapat mengakses rute panel admin.
  * [index.blade.php (Users Admin)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/admin/users/index.blade.php): Tabel daftar manajemen user.
* **Fitur Tarif Listrik PLN (Admin):**
  * [ElectricityRateController.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Http/Controllers/Admin/ElectricityRateController.php): Resource controller (CRUD) pengelolaan tarif per kWh.
  * [ElectricityRate.php](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/app/Models/ElectricityRate.php): Model database untuk mencatat relasi daya VA dan tarif per kWh.
  * [index.blade.php (Rates Admin)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/resources/views/admin/electricity_rates/index.blade.php): Tabel konfigurasi tarif per kWh berdasarkan daya VA.

---

## 🛣️ Konfigurasi Rute Aplikasi (Routing)
Semua rute di atas dideklarasikan di dalam file routing utama Laravel:
* [web.php (Routes)](file:///c:/Users/Lenovo/Documents/PPL/wattcare%20-%20Copy%20%282%29/routes/web.php)
  * Mengatur grup middleware `auth` untuk fitur pengguna dan prefix `admin` beserta middleware `AdminMiddleware` untuk fitur administratif.
