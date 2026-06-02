# Mapping PBI ke File (WattCare)

Dokumen ini memetakan Product Backlog Item (PBI) ke file yang digunakan pada aplikasi WattCare.

| PBI | Ringkas | File Terkait |
| --- | --- | --- |
| PBI-01 | Sidebar navigation | resources/views/partials/sidebar.blade.php, resources/views/layouts/app.blade.php, routes/web.php |
| PBI-02 | Alert lonjakan konsumsi | app/Services/AlertService.php, app/Http/Controllers/DashboardController.php, resources/views/dashboard/index.blade.php, config/constants.php |
| PBI-03 | Ringkasan energi | app/Http/Controllers/DashboardController.php, resources/views/dashboard/index.blade.php |
| PBI-04 | Monitoring emisi CO2 | app/Services/CalculatorService.php, app/Http/Controllers/AnalysisController.php, app/Http/Controllers/DashboardController.php, resources/views/dashboard/index.blade.php |
| PBI-05 | Grafik konsumsi harian | app/Http/Controllers/DashboardController.php, resources/views/dashboard/index.blade.php, resources/js/app.js |
| PBI-06 | Input tanggal | app/Http/Requests/AnalysisRequest.php, app/Http/Controllers/AnalysisController.php, resources/views/analysis/input.blade.php |
| PBI-07 | Input daya listrik (VA) | app/Http/Requests/RegisterRequest.php, app/Http/Requests/ProfileUpdateRequest.php, resources/views/auth/register.blade.php, resources/views/profile/edit.blade.php, app/Models/User.php |
| PBI-08 | Tambah data elektronik | app/Http/Controllers/DeviceController.php, app/Http/Requests/DeviceRequest.php, resources/views/devices/create.blade.php |
| PBI-09 | Simpan data elektronik | app/Http/Controllers/DeviceController.php, app/Models/Device.php, database/migrations/2024_01_01_000001_create_devices_table.php |
| PBI-10 | Filter riwayat | app/Http/Controllers/HistoryController.php, resources/views/history/index.blade.php |
| PBI-11 | Reset filter | resources/views/history/index.blade.php |
| PBI-12 | Update ringkasan dinamis | app/Http/Controllers/HistoryController.php, resources/views/history/index.blade.php |
| PBI-13 | Informasi tarif listrik | app/Models/User.php, config/constants.php |
| PBI-14 | Grafik konsumsi jangka panjang | app/Http/Controllers/DashboardController.php, resources/views/dashboard/index.blade.php |
| PBI-15 | Banner penghematan | resources/views/dashboard/index.blade.php, app/Services/RecommendationService.php |
| PBI-16 | Tips hemat energi | app/Services/RecommendationService.php, resources/views/dashboard/index.blade.php, resources/views/recommendations/index.blade.php |
| PBI-17 | Detail tips energi | resources/views/recommendations/index.blade.php |
| PBI-18 | Sinkronisasi data real-time | app/Http/Controllers/AnalysisController.php, app/Http/Controllers/DeviceController.php, app/Http/Controllers/HistoryController.php |
| PBI-19 | Kelola data elektronik | app/Http/Controllers/DeviceController.php, resources/views/devices/index.blade.php21, resources/views/devices/edit.blade.php |
| PBI-20 | Input durasi penggunaan | app/Http/Requests/AnalysisRequest.php, app/Http/Controllers/AnalysisController.php, resources/views/analysis/input.blade.php |
| PBI-21 | Perhitungan konsumsi otomatis | app/Services/CalculatorService.php, app/Http/Controllers/AnalysisController.php | MonitoringLog.php
