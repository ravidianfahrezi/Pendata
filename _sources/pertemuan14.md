# Pertemuan 14

## Peramalan Kadar NO₂ di Daerah Bangkalan Menggunakan KNN Regression

## Latar Belakang

Peningkatan aktivitas industri, transportasi, serta pertumbuhan populasi yang pesat telah menyebabkan peningkatan signifikan terhadap tingkat pencemaran udara di berbagai wilayah. Salah satu polutan udara utama yang menjadi perhatian adalah Nitrogen Dioksida (NO₂), yaitu gas beracun yang dihasilkan terutama dari proses pembakaran bahan bakar fosil seperti kendaraan bermotor, pembangkit listrik, dan kegiatan industri.

NO₂ memiliki dampak serius terhadap kesehatan manusia, seperti gangguan pernapasan, iritasi paru-paru, serta memperburuk penyakit asma dan bronkitis. Selain itu, NO₂ juga berkontribusi terhadap pembentukan hujan asam dan penurunan kualitas lingkungan secara keseluruhan.

Pada penelitian ini dilakukan analisis data time series harian kadar NO₂ di wilayah Bangkalan, Madura. Data diperoleh dari satelit Sentinel-5P melalui platform Copernicus Data Space Ecosystem. Setelah dilakukan preprocessing, data digunakan untuk membangun model prediksi menggunakan algoritma K-Nearest Neighbor (KNN) Regression.

---

# 1. Pengumpulan Data

Data NO₂ diperoleh dari platform Copernicus Data Space Ecosystem menggunakan dataset Sentinel-5P Level-2.

Sumber Data:

https://dataspace.copernicus.eu/

Dataset yang digunakan:

* Sentinel-5P Level-2
* Band: NO₂
* Periode: 1 Oktober 2023 – 30 September 2025
* Lokasi: Bangkalan, Madura

Proses pengambilan data dilakukan menggunakan library OpenEO pada lingkungan JupyterLab Copernicus.

Data yang diunduh disimpan dalam format NetCDF (`.nc`) yang berisi data spasial-temporal hasil pengamatan satelit.

---

# 2. Preprocessing Data

## 2.1 Ekstraksi Data NO₂

File NetCDF berisi beberapa variabel, yaitu:

* t (waktu)
* x (koordinat longitude)
* y (koordinat latitude)
* crs
* NO2

Variabel NO₂ memiliki bentuk data tiga dimensi:

```python
(725, 9, 8)
```

Keterangan:

* 725 hari pengamatan
* 9 baris grid spasial
* 8 kolom grid spasial

Setiap hari memiliki matriks data berukuran 9 × 8.

---

## 2.2 Penanganan Missing Value

Pada data NO₂ ditemukan beberapa nilai kosong (missing value) yang ditandai dengan simbol:

```text
--
```

Untuk mengatasi permasalahan tersebut digunakan metode Linear Interpolation pada setiap grid spasial.

Metode ini mengisi nilai yang hilang berdasarkan hubungan linear antara data sebelum dan sesudahnya sehingga kontinuitas data tetap terjaga.

---

## 2.3 Rata-rata Data Harian

Setelah missing value diperbaiki, setiap matriks 9 × 8 dirata-ratakan sehingga diperoleh satu nilai NO₂ untuk setiap hari.

Contoh hasil:

| Date       | NO₂      |
| ---------- | -------- |
| 2023-10-01 | 0.000027 |
| 2023-10-02 | 0.000024 |
| 2023-10-03 | 0.000024 |
| 2023-10-04 | 0.000021 |
| 2023-10-05 | 0.000021 |

Data kemudian disimpan dalam format CSV.

---

## 2.4 Pengecekan Missing Date

Setelah data berbentuk time series harian, dilakukan pengecekan kelengkapan tanggal.

Hasil pengecekan menunjukkan terdapat 6 tanggal yang hilang:

```text
2023-11-11
2024-01-01
2024-03-23
2024-08-12
2025-01-30
2025-01-31
```

Tanggal yang hilang tersebut diisi menggunakan metode Linear Interpolation berbasis waktu.

Setelah proses interpolasi:

```text
Jumlah missing date: 0
```

Jumlah data akhir:

```text
731 record
```

---

## 2.5 Deteksi Outlier Menggunakan IQR

Deteksi outlier dilakukan menggunakan metode Interquartile Range (IQR).

### Hasil Perhitungan

```text
Q1 = 1.9059601982007734e-05
Q3 = 2.886301626858767e-05
IQR = 9.803414286579937e-06

Lower Bound = 4.354480552137829e-06
Upper Bound = 4.3568137698457576e-05
```

Jumlah outlier yang terdeteksi:

```text
14 data
```

Outlier kemudian diubah menjadi NaN dan diisi kembali menggunakan Linear Interpolation.

Setelah proses interpolasi:

```text
Sisa outlier: 1
```

---

# 3. Modeling Menggunakan KNN Regression

## 3.1 Normalisasi Data

Karena algoritma KNN sensitif terhadap skala data, dilakukan normalisasi menggunakan Min-Max Scaler.

Rentang nilai setelah normalisasi:

```text
0 sampai 1
```

Contoh data:

| Date       | NO₂      | NO₂ Scaled |
| ---------- | -------- | ---------- |
| 2023-10-01 | 0.000027 | 0.238203   |
| 2023-10-02 | 0.000024 | 0.192840   |
| 2023-10-03 | 0.000024 | 0.196854   |
| 2023-10-04 | 0.000021 | 0.149560   |
| 2023-10-05 | 0.000021 | 0.154247   |

---

## 3.2 Uji Korelasi

Data time series diubah menjadi supervised learning menggunakan metode lag.

Pengujian korelasi dilakukan pada 30 hari sebelumnya.

Hasil korelasi tertinggi diperoleh pada:

| Lag | Korelasi |
| --- | -------- |
| t-1 | 0.796428 |
| t-2 | 0.675922 |
| t-3 | 0.593804 |
| t-4 | 0.523747 |

Karena seluruh nilai di atas 0.5, maka digunakan sebagai fitur utama dalam pemodelan.

---

## 3.3 Transformasi Data Supervised

### Lag 4

Jumlah data:

```text
(727, 5)
```

Fitur:

```text
NO2(t-4)
NO2(t-3)
NO2(t-2)
NO2(t-1)
```

Label:

```text
NO2(t)
```

---

### Lag 10

Jumlah data:

```text
(721, 11)
```

---

### Lag 30

Jumlah data:

```text
(701, 31)
```

---

# 4. Evaluasi Model

Pembagian data:

* Training: 80%
* Testing: 20%

Parameter KNN:

```python
n_neighbors = 5
```

## 4.1 Lag 4 Hari Sebelumnya

```text
Train Size: 581
Test Size : 146

RMSE : 0.065457
R²    : 0.1399
MAPE  : 61.0797%
```

---

## 4.2 Lag 10 Hari Sebelumnya

```text
Train Size: 576
Test Size : 145

RMSE : 0.067596
R²    : 0.0888
MAPE  : 64.6648%
```

---

## 4.3 Lag 30 Hari Sebelumnya

```text
Train Size: 560
Test Size : 141

RMSE : 0.074653
R²    : -0.0820
MAPE  : 71.9605%
```

---

# 5. Visualisasi

## 5.1 Deteksi Outlier

Tambahkan gambar hasil deteksi outlier di sini.

```markdown
![Deteksi Outlier](images/pertemuan14/outlier_iqr.png)
```

---

## 5.2 Data Setelah Interpolasi

Tambahkan gambar hasil interpolasi data.

```markdown
![Interpolasi Data](images/pertemuan14/interpolasi.png)
```

---

## 5.3 KNN Regression Lag 4

```markdown
![KNN Lag 4](images/pertemuan14/knn_lag4.png)
```

---

## 5.4 KNN Regression Lag 10

```markdown
![KNN Lag 10](images/pertemuan14/knn_lag10.png)
```

---

## 5.5 KNN Regression Lag 30

```markdown
![KNN Lag 30](images/pertemuan14/knn_lag30.png.png)
```

---

# 6. Kesimpulan

Berdasarkan hasil penelitian yang telah dilakukan, model KNN Regression mampu digunakan untuk melakukan prediksi kadar NO₂ harian di wilayah Bangkalan. Hasil evaluasi menunjukkan bahwa model dengan 4 hari sebelumnya memberikan performa terbaik dibandingkan model dengan 10 dan 30 hari sebelumnya.

Model lag 4 menghasilkan nilai RMSE sebesar 0.065457, R² sebesar 0.1399, dan MAPE sebesar 61.0797%. Ketika jumlah lag ditingkatkan menjadi 10 dan 30 hari sebelumnya, performa model justru mengalami penurunan yang ditunjukkan oleh meningkatnya nilai RMSE dan MAPE serta menurunnya nilai R².

Hasil tersebut menunjukkan bahwa penambahan jumlah fitur historis tidak selalu meningkatkan performa model KNN. Pada kasus data NO₂ Bangkalan, penggunaan empat hari sebelumnya sudah cukup representatif dibandingkan penggunaan riwayat yang lebih panjang.

Secara keseluruhan, model KNN masih menghasilkan tingkat kesalahan yang cukup tinggi sehingga diperlukan penelitian lanjutan menggunakan metode lain seperti Random Forest, XGBoost, LSTM, atau GRU untuk memperoleh hasil prediksi yang lebih baik.
