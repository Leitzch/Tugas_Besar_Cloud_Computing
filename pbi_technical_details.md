# Detail Teknis & Potongan Kode (Code Snippets) Per PBI

Dokumen ini menyajikan rincian logika pemrograman, alur kerja, dan potongan kode (*code snippets*) nyata untuk setiap **Product Backlog Item (PBI)** pada aplikasi **WattCare**. Dokumen ini siap dibagikan kepada tim pengembang untuk mempermudah proses peninjauan kode (*code review*) dan integrasi sistem.

---

## 🔌 1. Fitur Pengingat (Reminder Feature)

### PBI-1.1: Pembuatan & Pengelolaan Timer di Sisi Klien
* **Logika Kerja:** Menggunakan objek state JavaScript global `timers` yang disinkronkan secara otomatis ke `localStorage` agar tidak hilang ketika halaman dimuat ulang.
* **Potongan Kode Utama (`resources/views/reminder/index.blade.php`):**
  ```javascript
  // Inisialisasi: memuat timer dari localStorage saat halaman siap
  const savedState = localStorage.getItem('wattcare_timers_list');
  if (savedState) {
      timers = JSON.parse(savedState);
      renderTimers();
      startGlobalClock();
  }

  // Event handler saat timer baru ditambahkan melalui form
  form.addEventListener('submit', function(e) {
      e.preventDefault();
      const duration = parseFloat(durationInput.value);
      const unit = unitSelect.value;
      const selectedDeviceId = deviceSelect.value;
      const selectedDeviceName = deviceSelect.options[deviceSelect.selectedIndex].getAttribute('data-name');
      
      let seconds = unit === 'minutes' ? duration * 60 : duration * 3600;
      let hoursForLog = unit === 'minutes' ? duration / 60 : duration;

      const newTimer = {
          id: 'timer_' + Date.now() + Math.random().toString(36).substr(2, 5),
          deviceId: selectedDeviceId,
          deviceName: selectedDeviceName,
          endTime: Date.now() + (seconds * 1000),
          hoursForLog: hoursForLog,
          originalDuration: duration,
          originalUnit: unit === 'minutes' ? 'Menit' : 'Jam',
          isFinished: false,
          notified: false
      };
      
      timers.push(newTimer);
      saveTimers();
      renderTimers();
      startGlobalClock();
  });
  ```

### PBI-1.2: Sistem Alarm & Push Notification
* **Logika Kerja:** Pemicuan suara alarm berulang (*looping audio*) didampingi penayangan modal visual terpusat serta pemanggilan antarmuka Notification API bawaan browser modern.
* **Potongan Kode Utama (`resources/views/reminder/index.blade.php`):**
  ```javascript
  // Menampilkan modal dialog dan memutar suara alarm
  function showModal(deviceName) {
      const sound = document.getElementById('timerSound');
      if (timerModal.style.display === 'flex') {
          modalQueue.push(deviceName);
      } else {
          modalDeviceName.innerText = deviceName;
          timerModal.style.display = 'flex';
          
          sound.currentTime = 0;
          sound.loop = true;
          sound.play().catch(error => console.log('Autoplay dicegah browser:', error));
      }
  }

  // Notifikasi desktop sistem operasi via Browser API
  if (Notification.permission === "granted") {
      new Notification("Waktu Penggunaan Habis!", {
          body: `Alat ${timer.deviceName} telah selesai digunakan.`,
          icon: '/favicon.ico'
      });
  }
  ```

### PBI-1.3: Pencatatan Log Konsumsi ke Database
* **Logika Kerja:** Mengirimkan data via AJAX POST ke endpoint `/reminder` yang memproses perhitungan kWh, biaya Rupiah sesuai golongan tarif PLN, dan konversi emisi karbon CO2 di backend.
* **Potongan Kode Utama (`app/Http/Controllers/ReminderController.php`):**
  ```php
  public function store(Request $request)
  {
      $request->validate([
          'device_id' => 'required|exists:devices,id',
          'jam_pemakaian' => 'required|numeric|min:0.01',
      ]);

      $device = Device::where('user_id', auth()->id())
          ->where('id', $request->input('device_id'))
          ->firstOrFail();

      // Perhitungan kWh menggunakan CalculatorService
      $kwh = $this->calculatorService->calculateKwh(
          (float) $device->daya_watt,
          (float) $request->input('jam_pemakaian'),
          (int) $device->jumlah_unit
      );

      // Simpan log dasar pemakaian
      $log = MonitoringLog::create([
          'user_id' => auth()->id(),
          'device_id' => $device->id,
          'tanggal' => now()->toDateString(),
          'jam_pemakaian' => $request->input('jam_pemakaian'),
          'total_kwh' => $kwh,
      ]);

      // Hitung dan simpan data billing serta emisi CO2
      $tariff = auth()->user()->tariff_per_kwh;
      $cost = $this->calculatorService->calculateCost($kwh, $tariff);
      $co2 = $this->calculatorService->calculateCo2($kwh);

      Billing::create([
          'log_id' => $log->id,
          'estimasi_biaya' => $cost,
          'tarif_per_kwh' => $tariff,
      ]);

      CO2Impact::create([
          'log_id' => $log->id,
          'emisi_co2' => $co2,
          'faktor_emisi' => (float) config('constants.co2_factor'),
      ]);

      return response()->json([
          'status' => 'success',
          'message' => 'Penggunaan alat ' . $device->nama_device . ' telah dicatat.'
      ]);
  }
  ```

---

## 📰 2. Fitur Berita (News Feature)

### PBI-2.1: Agregasi Berita SDG 7 Terintegrasi & Caching
* **Logika Kerja:** Backend memanggil API eksternal `newsapi.org`, memilah teks menggunakan regex pencocokan kata kunci bertema SDG 7, menyortir teks non-Latin, dan menyimpannya di cache selama 1 jam.
* **Potongan Kode Utama (`app/Services/NewsService.php`):**
  ```php
  $cacheKey = 'sdg7_news_articles_' . md5($apiKey . '|' . $lang);
  $articles = Cache::remember($cacheKey, 3600, function () use ($apiKey) {
      // Pemanggilan HTTP Client ke NewsAPI
      $response = Http::timeout(10)->get('https://newsapi.org/v2/everything', [
          'q' => '"renewable energy" OR "solar panel" OR "transisi energi" OR "SDG 7"',
          'sortBy' => 'publishedAt',
          'apiKey' => $apiKey,
      ]);

      if ($response->successful()) {
          // Melakukan pemetaan array artikel berita hasil respon...
      }
      return $this->getMockNews();
  });

  // Pemfilteran bahasa/skrip dan relevansi strict SDG 7
  $filteredArticles = [];
  foreach ($articles as $article) {
      // Abaikan artikel dengan aksara Arab, Mandarin, Jepang, Korea, dsb.
      if (preg_match('/[\p{Cyrillic}\p{Arabic}\p{Han}\p{Hiragana}\p{Katakana}\p{Hangul}]/u', $article['title'])) {
          continue;
      }
      // Pengecekan kata kunci SDG 7
      $textToSearch = Str::lower($article['title'] . ' ' . $article['description']);
      foreach ($this->sdg7FilterKeywords as $keyword) {
          if (Str::contains($textToSearch, Str::lower($keyword))) {
              $filteredArticles[] = $article;
              break;
          }
      }
  }
  ```

### PBI-2.2: Mock News Fallback
* **Logika Kerja:** Jika API Key eksternal kosong/belum dikonfigurasi, sistem memanggil data statis lokal untuk mencegah kegagalan program.
* **Potongan Kode Utama (`app/Services/NewsService.php`):**
  ```php
  public function getMockNews(): array
  {
      return [
          [
              'id' => '1',
              'title' => 'Transisi Energi Bersih: Pemerintah Dorong Pemasangan Solar Panel di Berbagai Sektor',
              'author' => 'Budi Santoso',
              'source' => 'WattCare News',
              'description' => 'Pemerintah Indonesia terus berupaya meningkatkan bauran energi terbarukan melalui gerakan nasional pemasangan solar panel...',
              'content' => "Jakarta, WattCare – Dalam rangka mempercepat pencapaian SDG Target 7...",
              'url' => 'https://www.esdm.go.id',
              'urlToImage' => 'https://images.unsplash.com/photo-1509391366360-2e959784a276',
              'publishedAt' => '21 Mei 2026',
              'is_mock' => true,
          ],
          // Berita mock tambahan...
      ];
  }
  ```

### PBI-2.3: Detail Tampilan Artikel Berita
* **Logika Kerja:** Melakukan pencarian spesifik artikel berdasarkan ID unik dari kumpulan koleksi hasil olahan.
* **Potongan Kode Utama (`app/Http/Controllers/NewsController.php`):**
  ```php
  public function show(string $id): View
  {
      $articles = $this->newsService->getSdg7News(15);
      
      // Menggunakan Laravel Collection helper to find by ID
      $article = collect($articles)->firstWhere('id', $id);

      if (!$article) {
          abort(404, 'Berita tidak ditemukan.');
      }

      return view('news.show', compact('article'));
  }
  ```

---

## 🤖 3. Fitur Chat AI (AI Chat Feature)

### PBI-3.1: Ekstraksi Data Perangkat dengan AI
* **Logika Kerja:** Memberikan prompt khusus yang menuntut API Gemini mengembalikan format JSON murni yang berisi struktur parameter perangkat elektronik.
* **Potongan Kode Utama (`app/Http/Controllers/ChatController.php`):**
  ```php
  $apiKey = env('GEMINI_API_KEY');
  $prompt = "Tugas Anda adalah mengekstrak data perangkat listrik dari kalimat user berikut, dan mengembalikannya HANYA dalam format JSON valid (tanpa blok markdown atau teks lain). Format JSON yang diharapkan adalah array of objects dengan struktur key: 'nama_perangkat' (string), 'jumlah' (int), 'daya_watt' (int), 'waktu_jam' (float). JIKA ada informasi parameter yang TIDAK disebutkan secara spesifik oleh user, JANGAN buat estimasi, melainkan isi value parameter tersebut dengan null. Jika tidak ada perangkat listrik sama sekali di kalimat, kembalikan array kosong [].\n\nKalimat: \"{$message}\"";

  $response = Http::post("https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent?key={$apiKey}", [
      'contents' => [
          ['parts' => [['text' => $prompt]]]
      ],
  ]);
  ```

### PBI-3.2: Validasi Kelengkapan Parameter Obrolan AI
* **Logika Kerja:** Program menelusuri data JSON hasil pemrosesan AI, mengumpulkan properti bernilai `null` dan menyusun tanggapan meminta kelengkapan info.
* **Potongan Kode Utama (`app/Http/Controllers/ChatController.php`):**
  ```php
  $devices = json_decode($aiText, true);
  $hasMissing = false;
  $missingInfoText = "";

  foreach ($devices as $device) {
      $nama = $device['nama_perangkat'] ?? null;
      $jumlah = $device['jumlah'] ?? null;
      $daya = $device['daya_watt'] ?? null;
      $waktu = $device['waktu_jam'] ?? null;

      $kurang = [];
      if ($nama === null || trim($nama) === '') $kurang[] = 'nama perangkat';
      if ($jumlah === null) $kurang[] = 'jumlah unit';
      if ($daya === null) $kurang[] = 'daya (Watt)';
      if ($waktu === null) $kurang[] = 'waktu (Jam)';

      if (!empty($kurang)) {
          $hasMissing = true;
          $namaDisplay = $nama ? $nama : 'Perangkat yang tidak disebutkan';
          $missingInfoText .= "- Untuk **{$namaDisplay}**, informasi **" . implode(', ', $kurang) . "** belum lengkap.\n";
      }
  }

  if ($hasMissing) {
      $finalReply = "Tolong lengkapi data berikut agar saya bisa menghitung estimasinya dengan akurat:\n\n" . $missingInfoText;
      return response()->json([
          'reply' => nl2br(preg_replace('/\*\*(.*?)\*\*/', '<b>$1</b>', e($finalReply)))
      ]);
  }
  ```

### PBI-3.3: Kalkulasi Estimasi Konsumsi kWh Dinamis
* **Logika Kerja:** Jika data lengkap, backend menghitung rumus fisika energi secara dinamis dan memformat hasilnya dengan tag HTML tebal (`<b>`).
* **Potongan Kode Utama (`app/Http/Controllers/ChatController.php`):**
  ```php
  $totalKwh = 0;
  $replyText = "Berdasarkan pertanyaan Anda, berikut adalah estimasi penggunaan listrik:\n\n";

  foreach ($devices as $device) {
      $kwh = ($device['jumlah'] * $device['daya_watt'] * $device['waktu_jam']) / 1000;
      $totalKwh += $kwh;
      
      $replyText .= "- **{$device['nama_perangkat']}**: {$device['jumlah']} unit, {$device['daya_watt']} Watt, selama {$device['waktu_jam']} jam = **" . number_format($kwh, 2, ',', '.') . " kWh**\n";
  }

  $replyText .= "\n**Total Konsumsi Listrik Estimasi:** **" . number_format($totalKwh, 2, ',', '.') . " kWh**";

  return response()->json([
      'reply' => nl2br(preg_replace('/\*\*(.*?)\*\*/', '<b>$1</b>', e($replyText)))
  ]);
  ```

---

## 👤 4. Fitur Profile & Admin (Profile/Admin Feature)

### PBI-4.1: Pengelolaan Profil & Validasi Daya VA
* **Logika Kerja:** Memastikan data profil yang diinput aman dan kapasitas daya VA tervalidasi minimal 450 VA di dalam form request.
* **Potongan Kode Utama (`app/Http/Requests/ProfileUpdateRequest.php`):**
  ```php
  public function rules(): array
  {
      $userId = $this->user() ? $this->user()->id : null;

      return [
          'name' => ['required', 'string', 'max:100'],
          'email' => ['required', 'email', 'max:100', 'unique:users,email,' . $userId],
          'daya_va' => ['required', 'integer', 'min:450'],
          'password' => ['nullable', 'string', 'min:6', 'confirmed'],
      ];
  }
  ```

### PBI-4.2: Administrasi Manajemen Pengguna (User CRUD)
* **Logika Kerja:** Logika standard CRUD panel admin untuk menambah, mengubah profil orang lain, mendaftarkan admin baru, atau menghapus pengguna.
* **Potongan Kode Utama (`app/Http/Controllers/Admin/UserController.php`):**
  ```php
  public function update(Request $request, User $user)
  {
      $data = $request->validate([
          'name' => 'required|string|max:255',
          'email' => 'required|email|unique:users,email,'.$user->id,
          'password' => 'nullable|min:6|confirmed',
          'role' => 'required|in:admin,user',
      ]);

      if (! empty($data['password'])) {
          $data['password'] = Hash::make($data['password']);
      } else {
          unset($data['password']);
      }

      $user->update($data);
      return redirect()->route('admin.users.index')->with('success','User updated');
  }
  ```

### PBI-4.3: Pengaturan Dinamis Tarif Listrik per kWh
* **Logika Kerja:** Mengatur konfigurasi tarif per kWh berdasarkan daya VA yang bersifat unik menggunakan model `ElectricityRate`.
* **Potongan Kode Utama (`app/Http/Controllers/Admin/ElectricityRateController.php`):**
  ```php
  public function store(Request $request)
  {
      $data = $request->validate([
          'daya_va' => 'required|integer|unique:electricity_rates,daya_va',
          'tarif_per_kwh' => 'required|numeric|min:0',
      ]);

      ElectricityRate::create($data);

      return redirect()->route('admin.electricity_rates.index')->with('success','Tarif listrik ditambahkan');
  }
  ```
