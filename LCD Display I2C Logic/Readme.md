# 🖥️ Logika & Cara Kerja LCD I2C - Lab Safety System

Dokumen ini menjelaskan mekanisme bagaimana LCD 16x2 dengan modul I2C bekerja dalam sistem keamanan laboratorium ini, baik dari sisi perangkat keras maupun alur logika pemrogramannya.

## 1. Arsitektur Komunikasi I2C
Berbeda dengan koneksi LCD paralel standar yang membutuhkan 6-10 kabel, sistem ini menggunakan protokol **I2C (Inter-Integrated Circuit)** yang hanya memerlukan 2 jalur komunikasi utama:

* **SDA (Serial Data Line):** Jalur untuk mengirimkan data karakter dan instruksi.
* **SCL (Serial Clock Line):** Jalur sinkronisasi data antara Arduino dan LCD.

**Alamat I2C:** Kode ini menggunakan alamat `0x27`. Modul I2C di belakang LCD bertindak sebagai *expander* yang menerjemahkan data serial dari Arduino menjadi sinyal paralel untuk menggerakkan kristal cair pada layar.

---

## 2. Hierarki Tampilan (Layouting)
Layar LCD dibagi menjadi dua baris statis dan dinamis dengan koordinat $(x, y)$:

| Baris | Koordinat | Konten | Sifat |
| :--- | :--- | :--- | :--- |
| **Baris 1** | `(0, 0)` | `Lab Safety Syst` | **Statis**: Sebagai identitas sistem. |
| **Baris 2** | `(0, 1)` | `Initializing` / `LOCKED` / `BAHAYA!` / `Aman` | **Dinamis**: Berubah sesuai status sensor & sistem. |

---

## 3. Alur Logika Pemrograman (Flowchart)



### A. Inisialisasi (Setup Phase)
1. `lcd.init()`: Memulai komunikasi dengan chip driver LCD.
2. `lcd.backlight()`: Mengaktifkan transistor pada modul I2C untuk menyalakan LED latar.
3. `lcd.clear()`: Membersihkan buffer memori agar tidak ada karakter sampah saat pertama kali menyala.

### B. Mekanisme Pembaruan Data (Loop Phase)
Program menggunakan logika **Conditional Update**. LCD tidak dibersihkan (`clear`) setiap saat untuk menghindari efek berkedip (*flicker*), melainkan menimpa karakter pada posisi yang sama.

#### 1. Logika Prioritas 1: System Lock
Jika `systemLock == true`, program akan langsung menuju koordinat `(0, 1)` dan menuliskan `System LOCKED`. Fungsi `return` mencegah kode di bawahnya berjalan, sehingga teks ini akan tetap terkunci di layar.

#### 2. Logika Prioritas 2: Threshold & Mute
Sistem mengevaluasi `nilaiSensor` terhadap `THRESHOLD`:
- **Kondisi Bahaya:** Menampilkan string `BAHAYA! Val:` diikuti variabel `nilaiSensor`.
- **Kondisi Mute:** Jika interupsi tombol aktif, teks berubah menjadi `Muted Val:`.
- **Kondisi Aman:** Menampilkan `Aman Val:`.

### C. Logika Penanganan Karakter (Anti-Ghosting)
Dalam kode terdapat penggunaan spasi tambahan setelah angka, contohnya:
```cpp
lcd.print(nilaiSensor);
lcd.print("    ");