# Perbandingan Global Histogram Equalization (GHE) dan Contrast Limited Adaptive Histogram Equalization (CLAHE) pada Citra Rontgen Dada, Bawah Laut, dan Jalan Malam Hari

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-4.13.0-green.svg)](https://opencv.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Completed-success.svg)]()

> **Penulis:** Muhamad Faridzqi Suryadi  
> **Program Studi:** Teknik Informatika, Fakultas Teknik dan Ilmu Komputer  
> **Institusi:** Universitas Nahdlatul Ulama Indonesia  
> **Dokumen Lengkap:** [Paper_CLAHE_Image_Enhancement.pdf](Paper_CLAHE_Image_Enhancement.pdf)

---

## 📌 Ringkasan Proyek

Peningkatan kontras merupakan salah satu **operasi dasar citra digital** untuk memunculkan detail tersembunyi pada citra dengan rentang dinamis sempit atau iluminasi yang tidak merata. Proyek ini membandingkan dua metode peningkatan kontras berbasis histogram:

1. **Global Histogram Equalization (GHE):** Meratakan sebaran intensitas secara global menggunakan satu fungsi kumulatif distribusi (CDF) tunggal.
2. **Contrast Limited Adaptive Histogram Equalization (CLAHE):** Membagi citra ke dalam blok kontekstual lokal (*tiles*), membatasi amplifikasi kontras menggunakan ambang batas potong (*clipLimit*), mendistribusikan ulang kelebihan piksel, dan menggabungkannya dengan interpolasi bilinear.

Pengujian dievaluasi secara komparatif pada **tiga domain citra dunia nyata**:
* **Medis:** Citra Rontgen Dada (*Chest X-Ray*) — Tantangan kontras rendah pada struktur tulang dan jaringan paru.
* **Lingkungan Bawah Laut:** Citra Terumbu Karang (*Underwater Coral*) — Tantangan kabut warna biru-hijau (*color cast*) akibat penyerapan spektrum cahaya merah oleh air.
* **Surveillance Malam Hari:** Citra Jalan Raya Malam (*Night Street*) — Tantangan bayangan gelap ekstrem berdampingan dengan sorot lampu lalu lintas (*glare*).

Untuk citra berwarna, ekualisasi diterapkan secara khusus pada **kanal $L^*$ di ruang warna CIE $L^*a^*b^*$** guna mencegah terjadinya pergeseran warna (*color hue shifting*).

---

## 📊 Hasil dan Rekapitulasi Metrik

Evaluasi objektif dinilai menggunakan tiga metrik tanpa referensi:
* **Shannon Entropy ($H$):** Mengukur kekayaan kandungan informasi dan detail tekstur citra.
* **Standar Deviasi ($\sigma$):** Mengukur sebaran dinamika kontras intensitas.
* **Absolute Mean Brightness Error (AMBE):** Mengukur pergeseran rata-rata kecerahan antara citra masukan dan keluaran.

### 1. Tabel Evaluasi Metrik Keseluruhan

| Domain Citra | Metode | Entropi ($H$) | Kontras ($\sigma$) | AMBE | Karakteristik Hasil |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Rontgen Dada** | Asli | 7.43 | 47.75 | 0.00 | Kontras rendah, struktur paru samar |
| | GHE | 7.34 | 73.55 | 15.12 | *Over-exposed*, histogram berbentuk sisir |
| | **CLAHE** | **7.68** | **55.13** | **7.65** | **Optimal:** Tulang rusuk tegas, detail vaskular jelas |
| **Bawah Laut** | Asli | 7.11 | 35.24 | 0.00 | Berkabut biru-kehijauan pekat |
| | GHE | 6.91 | 73.53 | 3.62 | Warna jenuh berlebih, tekstur karang kasar |
| | **CLAHE** | **7.69** | **51.79** | **3.33** | **Optimal:** Tekstur karang 3D tampak alami |
| **Jalan Malam** | Asli | 7.16 | 51.35 | 0.00 | Objek gedung tenggelam dalam bayangan |
| | GHE | 6.91 | 72.88 | 63.32 | Rusak: Langit malam memutih silau |
| | **CLAHE** | **7.67** | **61.51** | **25.05** | **Optimal:** Marka jalan & gedung terangkat, lampu stabil |

> **Temuan Kunci:** Nilai *Shannon Entropy* pada CLAHE secara konsisten meningkat di seluruh citra uji ($7.43 \rightarrow 7.68$, $7.11 \rightarrow 7.69$, $7.16 \rightarrow 7.67$), sedangkan pada GHE justru menurun karena sebagian level intensitas mengalami saturasi jenuh (putih/hitam ekstrem).

---

### 2. Uji Noise pada Area Homogen (*Patch Analysis*)

Untuk menguji apakah CLAHE benar-benar membatasi derau pada area seragam, diukur rata-rata dan standar deviasi pada dua area homogen:
* **Patch Pojok Rontgen** (baris 0–119, kolom 0–249)
* **Patch Langit Malam** (baris 0–249, kolom 900–1699, kanal L)

| Citra (Patch) | Metode | Rata-rata | Standar Deviasi ($\sigma$) |
| :--- | :--- | :---: | :---: |
| **Rontgen (Pojok)** | Asli | 4.80 | 2.90 |
| | GHE | 3.45 | 1.21 |
| | CLAHE | 12.30 | 4.49 |
| **Malam (Langit)** | Asli | 5.80 | 1.85 |
| | GHE | 15.32 | 10.13 |
| | **CLAHE** | **18.37** | **5.79** |

*Pada citra malam, CLAHE berhasil menahan kenaikan standar deviasi langit jauh lebih baik daripada GHE ($5.79$ vs $10.13$). Pada pojok rontgen, nilai rata-rata dan std CLAHE naik karena blok lokal yang hampir seluruhnya gelap diregangkan.*

---

### 3. Analisis Hyperparameter CLAHE

* **`clipLimit` (diuji pada grid $8 \times 8$ tetap):** Entropi meningkat dari $7.53$ (`clipLimit=1.0`) ke $7.83$ (`clipLimit=8.0`), lalu jenuh pada nilai 16.0. Namun pada `clipLimit > 4.0`, tekstur jaringan mulai tampak kasar sehingga nilai **$2.0 - 3.0$** merupakan titik keseimbangan terbaik (*sweet spot*).
* **`tileGridSize` (diuji pada `clipLimit=2.0` tetap):** Entropi menurun saat ukuran grid semakin rapat ($7.91$ pada $2\times2$ menjadi $7.51$ pada $16\times16$). Grid $2\times2$ secara metrik menghasilkan angka tertinggi namun secara visual sangat mirip dengan kelemahan GHE. Oleh karena itu, grid **$8 \times 8$** dipilih sebagai kompromi visual yang paling proporsional.

---


## 🛠️ Sistem Pustaka

* **Python:** `>= 3.10`
* **OpenCV (`opencv-python`):** `>= 4.8.0`
* **NumPy:** `>= 1.24.0`
* **Matplotlib:** `>= 3.7.0`
* **Pandas:** `>= 2.0.0`

---

## 📖 Referensi Utama

1. **Gonzalez, R. C., & Woods, R. E. (2018).** *Digital Image Processing* (4th ed.). Pearson Education.
2. **Zuiderveld, K. (1994).** *Contrast Limited Adaptive Histogram Equalization*. Graphics Gems IV, Academic Press Professional, Inc., pp. 474–485.
3. **Pizer, S. M., et al. (1987).** *Adaptive Histogram Equalization and Its Variations*. Computer Vision, Graphics, and Image Processing, 39(3), 355–368.
4. **Reza, A. M. (2004).** *Realization of the Contrast Limited Adaptive Histogram Equalization (CLAHE) for Real-Time Image Enhancement*. Journal of VLSI Signal Processing Systems, 38(1), 35–44.
5. **Shannon, C. E. (1948).** *A Mathematical Theory of Communication*. Bell System Technical Journal, 27(3), 379–423.
6. **Chen, S.-D., & Ramli, A. R. (2003).** *Minimum Mean Brightness Error Bi-Histogram Equalization in Contrast Enhancement*. IEEE Transactions on Consumer Electronics, 49(4), 1310–1319.

---

## 📄 Lisensi
Proyek ini didistribusikan di bawah lisensi akademik [MIT License](LICENSE) untuk keperluan studi dan penelitian dalam bidang Pengolahan Citra Digital.
