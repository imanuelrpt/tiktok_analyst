# Laporan Analisis Konten TikTok

**Pertanyaan utama:** jenis konten apa yang paling banyak mendapat interaksi?

Laporan portofolio Data Analyst. Semua angka diambil dari file CSV hasil eksekusi notebook Tahap 2-5. Jika data diubah dan notebook dijalankan ulang, angka di sini ikut berubah.

## 1. Ringkasan eksekutif

- Dataset **TikTok Video Metadata**, lisensi **Apache 2.0**, diambil dengan **upload manual** dari Kaggle.
- Setelah cleaning: **751 baris × 16 kolom**; 11 keputusan cleaning tercatat.
- Like rate median seluruh data **4,84% (CI 4,58% - 5,35%)**; like rate adalah proxy karena kolom komentar dan share tidak ada.
- Kelompok durasi **15-30 detik** punya like rate median **7,60%**, lebih tinggi daripada **≥60 detik** **4,00%**; uji peringkat tetap signifikan setelah koreksi.
- Namun pola durasi hanya konsisten pada **2 dari 5 tahun** yang cukup sampel, jadi klaimnya dibatasi.
- **8 video teratas** menyumbang **29,0% total views**; like rate median mereka 5,48% vs 4,84% pada sisanya.
- Pertanyaan tentang bahasa/wilayah dan jenis interaksi dominan **tidak bisa dijawab** karena kolomnya tidak ada.

## 2. Sumber data dan etika

| Item | Keterangan |
|---|---|
| Judul dataset | TikTok Video Metadata |
| URL Kaggle | https://www.kaggle.com/datasets/maratsaratov/tiktok-data |
| Lisensi | Apache 2.0 |
| Cara pengumpulan | Tidak dijelaskan rinci di halaman Kaggle; hanya disebut metadata video TikTok nyata dari berbagai topik dan wilayah |
| Rentang waktu | 2015-07-10 s/d 2025-06-14 (berdasarkan human_time yang berhasil di-parse) |
| Metode akuisisi | Upload manual, tanpa Kaggle API |

Dataset ini adalah **snapshot**, bukan data real-time. Deskripsi Kaggle menyebut metadata video dari berbagai topik dan wilayah, tetapi tidak ada kolom wilayah di file, jadi analisis Indonesia tidak mungkin dilakukan.

Privasi: **65 caption** disamarkan (tanda @, label kontak, nomor diganti penanda). Tidak ada nama akun, URL profil, atau kolom identitas yang disimpan. Caption mentah tidak dikutip di laporan ini. Penyamaran memakai pola, jadi nama orang yang tidak mengikuti pola mungkin tetap tertinggal — ini keterbatasan yang jujur.

## 3. Metode singkat (bahan interview)

- **Median, bukan mean:** distribusi views sangat miring (beberapa video sangat viral), sehingga mean tidak mewakili video umum.
- **like_rate = likes / views:** proxy interaksi. Definisi engagement_rate `(likes+comments+shares)/views` tidak bisa dihitung karena `comments` dan `shares` tidak tersedia. Kolom palsu tidak dibuat.
- **Outlier tidak dihapus:** video viral justru menjadi objek analisis. Penanggulangannya memakai median, persentil, log skala, perbandingan top 1%, dan uji berbasis peringkat.
- **Bootstrap 2.000 kali (seed 42):** mengukur rentang ketidakpastian median tanpa asumsi distribusi normal.
- **Kruskal-Wallis dan Mann-Whitney + Bonferroni:** uji beda antar kelompok berbasis peringkat, dengan koreksi karena tiga pasangan diuji.
- **Ambang frekuensi:** hashtag/sound minimal 5 video agar tidak didominasi satu video; klaim jam hanya untuk n ≥ 10.
- **Kategori konten:** tesaurus kata kunci buatan sendiri, bukan label resmi platform.

## 4. Temuan utama

### 4.1 Distribusi dan video paling viral

- Median views **394.800**, mean **5.661.243**. Perbedaan ini menunjukkan distribusi yang sangat condong.
- 8 video teratas menguasai **29,0% total views** dan 26,4% total likes.
- Interpretasi: sebagian kecil video menyumbang porsi besar tayangan, jadi ringkasan harus memakai median.

### 4.2 Durasi

| Durasi | n | Median views | Like rate median | CI bootstrap |
|---|---:|---:|---:|---|
| <15 detik | 164 | 506.550 | 6,55% | 5,70-7,50% |
| 15-30 detik | 133 | 646.600 | 7,60% | 6,31-8,95% |
| 30-60 detik | 93 | 559.300 | 5,75% | 4,61-6,74% |
| ≥60 detik | 345 | 241.200 | 4,00% | 3,74-4,36% |
| tidak diketahui | 16 | 364.450 | 6,96% | - |

- Uji Kruskal-Wallis dan perbandingan 15-30 vs ≥60 detik tetap signifikan setelah Bonferroni.
- Sensitivitas outlier: Perubahan maksimal 0,06 poin persen; Spearman durasi-like_rate -0,208.
- Kontrol follower: pola 15-30 > ≥60 konsisten di semua tier. Kontrol tahun: hanya **2 dari 5 tahun**.
- Interpretasi: durasi pendek berasosiasi dengan like_rate lebih tinggi pada sampel ini, tetapi efek tahun belum terkontrol penuh; jangan dinyatakan sebagai sebab-akibat.

### 4.3 Waktu posting

- Hari: Selasa **7,02%** (n=68), Rabu **3,82%** (n=236).
- Jam 22:00 punya **173 video** dengan like rate **3,59%**.
- Interpretasi: konsentrasi sampel pada jam tertentu dan tidak adanya zona waktu membuat hasil ini tidak layak dijadikan rekomendasi jam posting; kemungkinan besar artefak pengumpulan data.

### 4.4 Hashtag, audio, dan kata kunci

- Paling sering: **fyp** pada 228 video (like rate 6,76%).
- Like rate tertinggi (n ≥ 5): **summer** **14,23%** (n=11).
- Item yang muncul di lebih dari satu definisi ambang: **2 ID audio anonim** dan **csgo, trend**.
- Uji kata kunci caption dari daftar heuristik tidak menemukan kata yang muncul di minimal 5 video grup atas, jadi tidak dibuat klaim.
- Interpretasi: lift di atas 1 berarti item lebih sering muncul di kelompok like_rate tinggi; ini asosiasi, bukan bukti bahwa item itu menyebabkan like.

### 4.5 Follower

- Tier **≥1 jt follower**: like rate **7,26%** (n=99), median views 2.500.000.
- Spearman followers vs like_rate **0,100**, followers vs views **0,340**.
- Interpretasi: akun besar punya tayangan lebih banyak, tetapi like per tayangan tidak ikut naik kuat.

### 4.6 Kategori heuristik

- Kategori buatan sendiri hanya menjangkau **31.7%** video; sisanya (513 video) masuk **Lainnya**.
- Interpretasi: ini contoh metode pengelompokan, bukan kesimpulan utama dan bukan label resmi platform.

## 5. Jawaban 8 pertanyaan

| # | Pertanyaan | Status | Jawaban ringkas |
|---|---|---|---|
| 1 | Jenis konten dengan views/likes tertinggi | Sebagian | Hashtag paling sering fyp pada 228 video; like rate tertinggi summer 14,23% (n=11) |
| 2 | Durasi vs tingkat interaksi | Sebagian | 15-30 detik 7,60% vs ≥60 detik 4,00%; signifikan=True, tapi hanya 2 dari 5 tahun yang konsisten |
| 3 | Jam & hari posting | Bisa dengan syarat | Selasa 7,02% (n=68), Rabu 3,82% (n=236); jam 22 n=173 like rate 3,59% |
| 4 | Hashtag, sound, kata kunci caption | Sebagian | Item stabil di beberapa ambang: 2 ID audio anonim + csgo, trend; kata caption tidak ada yang mencapai ambang 5 video |
| 5 | Follower vs engagement rate | Sebagian | Tier ≥1 jt follower like rate 7,26% (n=99); Spearman followers vs like_rate 0,100 |
| 6 | Perbedaan bahasa/wilayah, fokus Indonesia | Tidak bisa dijawab | Tidak dijawab |
| 7 | Jenis interaksi dominan per konten | Tidak bisa dijawab | Tidak dijawab |
| 8 | Rekomendasi untuk kreator/brand | Bisa dengan keterbatasan | Uji eksperimen format 15-30 detik dan hashtag ber-like-rate tinggi; jangan pakai jam posting sebagai rekomendasi |

Detail batasan setiap jawaban ada di `outputs/06_jawaban_pertanyaan.csv`.

## 6. Keterbatasan

- Tidak ada `comments` dan `shares`, jadi engagement rate definisi awal tidak bisa dihitung.
- Tidak ada kolom bahasa, negara, wilayah, atau usia penonton. Tidak ada klaim tentang Gen Z, anak, orang tua, atau Indonesia.
- Data snapshot lintas 2015-2025; video lama punya waktu lebih lama mengumpulkan views (Spearman tahun vs like_rate -0,273).
- Zona waktu tidak tercantum, sehingga jam posting tidak bisa diartikan secara lokal.
- Kategori konten heuristik hanya menjangkau 31.7% video.
- Penyamaran caption berbasis pola, bukan jaminan 100% tanpa nama orang.
- Semua hubungan bersifat asosiasi; tidak ada desain eksperimen, jadi tidak ada klaim sebab-akibat.
- Cara pengumpulan dataset tidak dijelaskan rinci di halaman Kaggle; tidak saya karang.

## 7. Daftar grafik

| File | Judul | Sumbu X | Sumbu Y | Interpretasi |
|---|---|---|---|---|
| `04_distribusi_views.png` | Distribusi Views (Skala Log10) | log10(views) | Jumlah video | Mean tertarik ke kanan oleh sedikit video viral, sehingga median lebih mewakili video umum. |
| `04_durasi_vs_like_rate.png` | Like Rate Median per Kelompok Durasi | Kelompok durasi | Like rate median (%) | Durasi 15-30 detik memiliki like rate median tertinggi sekaligus median views tertinggi di antara kelompok berdurasi jelas. |
| `04_hari_posting.png` | Like Rate Median per Hari Posting | Hari posting | Like rate median (%) | Rabu punya like rate median terendah dengan sampel terbanyak, sedangkan Selasa tertinggi dengan sampel lebih sedikit. |
| `04_jam_posting.png` | Like Rate Median per Jam Posting | Jam posting (zona waktu tidak tercantum) | Like rate median (%) | Jam 22 menyumbang sampel paling banyak dengan like rate rendah, kemungkinan artefak pengumpulan data. |
| `04_top_hashtag.png` | Top 10 Hashtag berdasarkan Like Rate Median (n >= 5) | Like rate median (%) | Hashtag | Hashtag dengan like rate tinggi belum tentu paling sering dipakai. |
| `04_kategori_konten.png` | Like Rate Median per Kategori Konten (Heuristik Kata Kunci) | Like rate median (%) | Kategori (dari hashtag + caption) | Kategori ini tesaurus buatan sendiri dan cakupannya terbatas, bukan label resmi TikTok. |
| `04_follower_vs_like_rate.png` | Like Rate Median per Tingkat Follower | Jumlah follower akun | Like rate median (%) | Korelasi followers dengan like_rate lemah meski tier teratas punya median tertinggi. |
| `04_top1pct_vs_sisanya.png` | Perbandingan 1% Video Teratas (berdasarkan views) vs Sisanya | Grup | Views (median) dan like rate median (%) | Delapan video teratas menyumbang porsi besar total views dengan like rate median sedikit lebih tinggi. |
| `05_bootstrap_durasi.png` | Median Like Rate per Durasi dengan Interval Kepercayaan Bootstrap 95% | Like rate median (%) | Kelompok durasi | Interval 15-30 detik seluruhnya di atas interval ≥60 detik, jadi perbedaan sulit dijelaskan kebetulan sampel. |
| `05_sensitivitas_top1pct.png` | Sensitivitas Like Rate Median: Data Lengkap vs Tanpa 1% Video Teratas | Kelompok durasi | Like rate median (%) | Membuang delapan video viral mengubah median maksimal 0,06 poin persen. |

Inventaris ini juga tersimpan di `outputs/06_inventaris_grafik.csv`. Dashboard tidak dibuat dalam sesi ini.

## 8. Cara menjalankan

```python
# Di Google Colab, unggah folder project, lalu jalankan berurutan:
# 01_setup.ipynb -> 02_data_understanding.ipynb -> 03_data_cleaning.ipynb
# -> 04_eda.ipynb -> 05_analisis_lanjutan.ipynb -> 06_laporan_analitik.ipynb

# Pastikan BASE sesuai folder kerja (di Colab: /content) dan folder data/ berisi tiktok_data.csv.
```

Struktur folder: `data/` (dataset mentah, tidak di-commit), `notebooks/` (kode), `outputs/` (tabel hasil), `images/` (grafik), `laporan/` (laporan ini).

