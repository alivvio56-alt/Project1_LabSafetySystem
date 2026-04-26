## Logic Sensor Handling & Input Processing

Bagian ini menjelaskan bagaimana sistem memproses input dari sensor dan interaksi pengguna untuk menentukan status keamanan laboratorium secara *real-time*.

### 1. Mekanisme Input Analog (ADC)
Sistem menggunakan pin **A0** sebagai input analog untuk membaca nilai dari sensor (seperti sensor asap atau gas). 
* **Rentang Pembacaan:** Data mentah dibaca menggunakan fungsi `analogRead()` dengan resolusi 10-bit (nilai berkisar antara `0` hingga `1023`).
* **Threshold Dinamis:** Batas ambang bahaya (`THRESHOLD`) tidak bersifat statis. Pengguna dapat mengubah nilai ini secara langsung melalui perintah UART `set [angka]`, sehingga kalibrasi dapat dilakukan tanpa perlu melakukan *re-upload* kode.

### 2. Status Kontrol Sistem
Perilaku logika sistem diatur oleh dua variabel status (*state*) utama:
* **`systemLock`**: Jika aktif (`true`), sistem akan menghentikan seluruh pembacaan sensor dan mematikan semua *actuator* (LED & Buzzer). Mode ini berguna untuk keperluan *maintenance*.
* **`muted`**: Jika aktif (`true`), alarm suara (Buzzer) akan dimatikan sementara, namun indikator visual (LED Merah) tetap aktif jika kondisi masih dalam ambang bahaya.

### 3. Matriks Logika Kondisional
Sistem mengevaluasi kondisi berdasarkan hierarki prioritas sebagai berikut:

| Kondisi Sensor | Status Mute | Status Lock | Output LED | Output Buzzer | Keterangan |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Apapun | Apapun | **Active** | Semua OFF | OFF | Sistem terkunci (Mode Standby) |
| `> THRESHOLD` | **False** | Inactive | Merah ON | **ON (Sweep)** | **BAHAYA!** Alarm aktif |
| `> THRESHOLD` | **True** | Inactive | Merah ON | OFF | Bahaya terdeteksi, suara di-mute |
| `< THRESHOLD` | Apapun | Inactive | Hijau ON | OFF | Kondisi AMAN |

### 4. Interrupt Handling (External Input)
Untuk memastikan responsivitas yang cepat saat mematikan alarm (mute), sistem menggunakan **Hardware Interrupt** pada pin digital **2**:
* **Metode:** `attachInterrupt(digitalPinToInterrupt(pinButton), handleMute, FALLING);`
* **Fungsi:** Saat tombol ditekan (logika *Falling Edge*), fungsi `handleMute()` akan segera dieksekusi oleh prosesor tanpa menunggu antrean *looping* utama selesai. Ini menjamin sistem segera merespon input pengguna meskipun sedang menjalankan proses lain.

### 5. Komunikasi Serial (UART Interface)
Sistem menyediakan antarmuka kontrol melalui Serial Monitor (Baud Rate 9600) dengan perintah berikut:
* `on`: Mengaktifkan sistem kembali dan mereset status *mute*.
* `lock`: Mengunci sistem dan mematikan semua fungsi monitoring.
* `set [nilai]`: Memperbarui variabel `THRESHOLD` secara instan (contoh: `set 150`).
