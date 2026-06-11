# TPertemuan 15

### Tugas Online
Tahap awal guna menyiapkan semua peralatan kode yang dibutuhkan. Kita mengimpor `pandas` guna olah data, `matplotlib` guna grafik, `skforecast` guna model prediksi waktu, `lightgbm` sebagai mesin algoritma dasar, serta `shap` dan `sklearn` guna melakukan analisis transparansi keputusan model.

```python
# Libraries
# ==============================================================================
import pandas as pd
import matplotlib.pyplot as plt
import shap
from sklearn.inspection import permutation_importance
from sklearn.inspection import PartialDependenceDisplay
from lightgbm import LGBMRegressor
from skforecast.datasets import fetch_dataset
from skforecast.recursive import ForecasterRecursive
```

### Mengambil Dataset Mentah
Mengambil dataset historis `vic_electricity` langsung via fitur bawaan `skforecast`. Data ini berisi rekaman konsumsi listrik harian beserta suhu udara di wilayah Victoria, Australia. Kode `data.head(3)` dipakai guna melihat sekilas 3 baris pertama data.


```python
# Download data
# ==============================================================================
data = fetch_dataset(name="vic_electricity")
data.head(3)
```

### Mengubah Data Menjadi Skala Harian (Resampling)
Data asli yang terekam per 30 menit digabungkan menjadi skala harian (`'D'`). Total konsumsi listrik harian dijumlahkan (`'sum'`), sedangkan suhu udara harian diambil nilai rata-ratanya (`'mean'`) agar data lebih konsisten guna prediksi jangka panjang.

```python
# Aggregation to daily frequency
# ==============================================================================
data = data.resample('D').agg({'Demand': 'sum', 'Temperature': 'mean'})
data.head(3)
```

### Memisahkan Data Latih dan Data Uji (Split Train-Test)
Memisahkan data menjadi dua bagian: data hingga tanggal 21 Desember 2014 dipakai sebagai data latih (`data_train`) guna membangun model, dan data setelah tanggal tersebut dijadikan data uji (`data_test`) guna menguji keakuratan tebakan model.

```python
# Split train-test
# ==============================================================================
data_train = data.loc[: '2014-12-21']
data_test = data.loc['2014-12-22':]
```

### Inisialisasi dan Pelatihan Model Peramalan
Membangun objek prediksi menggunakan `ForecasterRecursive` dengan algoritma `LGBMRegressor`. Model ini dikonfigurasi guna melihat data 7 hari ke belakang (`lags=7`) dan memanfaatkan suhu (`exog`) guna menebak total konsumsi listrik di masa depan.

```python
# Create a recursive multi-step forecaster (ForecasterRecursive)
# ==============================================================================
forecaster = ForecasterRecursive(
                 estimator = LGBMRegressor(random_state=123, verbose=-1),
                 lags      = 7
             )

forecaster.fit(
    y    = data_train['Demand'],
    exog = data_train['Temperature']
)
forecaster
```

### Melihat Tingkat Kepentingan Fitur secara Global
Menjalankan perintah `forecaster.get_feature_importances()` guna melihat sekilas fitur atau variabel mana yang paling sering dipakai dan dianggap paling penting oleh model dalam menentukan keputusan prediksi secara umum.

```python
# Predictors importances
# ==============================================================================
forecaster.get_feature_importances()
```

### Mengekstrak Matriks Data Latih Internal
Memanfaatkan fungsi `create_train_X_y` guna membedah bentuk tabel data ($X$ dan $y$) yang dibuat secara otomatis oleh `skforecast` di latar belakang sebelum dimasukkan ke dalam algoritma pelatihan *machine learning*.

```python
# Training matrices used by the forecaster to fit the internal regressor
# ==============================================================================
X_train, y_train = forecaster.create_train_X_y(
                       y    = data_train['Demand'],
                       exog = data_train['Temperature']
                   )

display(X_train.head(3))
display(y_train.head(3))
```

### Menyiapkan Alat Bedah SHAP (Explainer)
Mengaktifkan fungsionalitas JavaScript lewat `shap.initjs()` dan membuat objek `TreeExplainer` yang diarahkan langsung ke mesin model kita (`forecaster.estimator`). Langkah ini penting guna menghitung kontribusi nilai (*SHAP values*) pada setiap baris data.

```python
# 1. Inisialisasi JS untuk SHAP
shap.initjs()

# 2. Ganti forecaster.regressor menjadi forecaster.estimator
explainer = shap.TreeExplainer(forecaster.estimator)

# 3. Hitung SHAP values
shap_values = explainer.shap_values(X_train)

```

### Grafik SHAP Summary Plot (Model Bar)
Memvisualisasikan grafik batang (*bar plot*) dari nilai SHAP guna mengurutkan variabel dari yang paling berpengaruh hingga yang kurang berpengaruh terhadap hasil prediksi konsumsi listrik secara keseluruhan.

```python

shap.summary_plot(shap_values, X_train, plot_type="bar")
```

### Grafik Distribusi Pengaruh Fitur (SHAP Density Plot)
Memvisualisasikan grafik titik SHAP guna melihat arah pengaruh variabel. Melalui grafik ini, kita bisa membaca apakah nilai suhu yang tinggi akan mendorong prediksi konsumsi listrik menjadi naik atau justru malah menurunkannya.

```python
shap.summary_plot(shap_values, X_train)
```

### Membedah Alasan Prediksi pada Baris Data Pertama
Memanfaatkan *Force Plot* guna menganalisis keputusan model secara spesifik (lokal) pada baris data pertama. Grafik ini akan menunjukkan faktor apa saja yang mendorong angka prediksi naik (merah) atau menahannya turun (biru) pada hari tersebut.

```python
shap.initjs()  # Tambahkan ini di sel yang sama
shap.force_plot(explainer.expected_value, shap_values[0, :], X_train.iloc[0, :])
```

### Visualisasi Kolektif Force Plot (200 Data Pertama)
Menggabungkan visualisasi *Force Plot* guna 200 baris data pertama sekaligus. Grafik interaktif ini sangat berguna guna melihat perubahan tren keputusan model seiring berjalannya waktu atau perubahan pola data.

```python
# Force plot for the first 200 observations in the training set
# ==============================================================================
shap.initjs()
shap.force_plot(explainer.expected_value, shap_values[:200, :], X_train.iloc[:200, :])
```

### Analisis Ketergantungan Variabel Suhu (Dependence Plot)
Membangun grafik khusus guna melihat hubungan linier atau non-linier antara fluktuasi variabel `Temperature` terhadap perubahan nilai prediksi, sekaligus mendeteksi interaksinya dengan variabel pendukung lain.

```python
# Dependence plot for Temperature
# ==============================================================================
fig, ax = plt.subplots(figsize=(7, 4))
shap.dependence_plot("Temperature", shap_values, X_train, ax=ax)
```

### Melakukan Peramalan Masa Depan (Predict)
Memerintahkan model yang telah dilatih guna melakukan prediksi konsumsi listrik sebanyak 10 langkah ke depan (`steps=10`) dengan memasukkan data prediktor suhu dari masa data uji (`data_test`).

```python
# Predict
# ==============================================================================
predictions = forecaster.predict(steps=10, exog=data_test['Temperature'])
predictions
```

### Membangun Matriks Input guna Proses Prediksi
Melihat bentuk matriks data ($X$) yang diatur secara otomatis oleh fungsi internal model sewaktu memproses langkah prediksi masa depan (tabel lag yang bergeser secara rekursif).

```python
# Create input matrix for predict method
# ==============================================================================
X_predict = forecaster.create_predict_X(steps=10, exog=data_test['Temperature'])
X_predict
```

### Membedah Alasan Hasil Ramalan Tanggal 22 Desember 2014
Menerapkan analisis *Force Plot* pada hasil tebakan masa depan guna tanggal spesifik ('2014-12-22'). Ini membantu memberikan pertanggungjawaban logis mengapa model meramal angka kebutuhan listrik sebesar itu pada tanggal tersebut.

```python
# Force plot for a specific prediction
# ==============================================================================
shap.initjs()
predicted_date = '2014-12-22'
iloc_predicted_date = X_predict.index.get_loc(predicted_date)
shap_values = explainer.shap_values(X_predict)
shap.force_plot(
    explainer.expected_value,
    shap_values[iloc_predicted_date, :],
    X_predict.iloc[iloc_predicted_date, :]
)
```

### Memuat Ulang Matriks Latih guna Evaluasi Lanjutan
Menyiapkan kembali pasangan data $X\_train$ dan $y\_train$ dari model guna mempersiapkan pengujian sensitivitas alternatif menggunakan fitur evaluasi bawaan dari `scikit-learn`.

```python
# Training matrices used by the forecaster to fit the internal regressor
# ==============================================================================
X_train, y_train = forecaster.create_train_X_y(
                       y    = data_train['Demand'],
                       exog = data_train['Temperature']
                   )

# Permutation importances
# ==============================================================================
r = permutation_importance(
    estimator    = forecaster.estimator,  # Ganti dari forecaster.regressor menjadi forecaster.estimator
    X            = X_train,
    y            = y_train,
    n_repeats    = 3,
    max_samples  = 0.5,
    random_state = 123
)

importances = pd.DataFrame({
    'feature': X_train.columns,
    'mean_importance': r.importances_mean,
    'std_importance': r.importances_std
}).sort_values('mean_importance', ascending=False)

importances
```

### Grafik Ketergantungan Parsial (Partial Dependence Plots)
Memvisualisasikan visualisasi akhir menggunakan modul `sklearn.inspection` guna mengukur efek marjinal dari satu atau dua fitur terpilih terhadap hasil prediksi model *decision tree*, sebagai validasi pelengkap dari hasil SHAP.

```python
# Scikit-learn partial dependence plots
# ==============================================================================
fig, ax = plt.subplots(figsize=(9, 4))
ax.set_title("Decision Tree")
pd.plots = PartialDependenceDisplay.from_estimator(
    estimator    = forecaster.estimator,
    X         = X_train,
    features  = ["Temperature", "lag_1"],
    kind      = 'both',
    ax        = ax,
)
ax.set_title("Partial Dependence Plot")
fig.tight_layout();
```

# Jawaban Pertanyaan

---

## 1. Analisis Prediksi tentang Apa?
Analisis ini membahas tentang **prediksi harian kebutuhan listrik (*daily electricity demand*)** di wilayah Victoria, Australia.

Selain memprediksi angka konsumsi di masa depan, fokus utamanya adalah **Model Explainability (Keterjelasan Model)**, yaitu membongkar cara kerja model *machine learning* (yang biasanya bersifat *black box*) guna memahami faktor apa saja yang paling memengaruhi naik-turunnya konsumsi listrik (misalnya cuaca atau pola hari sebelumnya).

---

## 2. Struktur Data Training (Input & Output)
Data mentah yang awalnya berskala per 30 menit digabungkan (*aggregated*) menjadi skala harian. Berikut adalah bentuk matriks input ($X$) dan output ($y$) yang dipakai guna melatih model:

| Jenis Variabel | Nama Kolom / Fitur | Deskripsi / Keterangan |
| :--- | :--- | :--- |
| **Output (Target / $y$)** | `Demand` | Total konsumsi listrik harian yang ingin diprediksi. |
| **Input (Features / $X$)** | `lag_1` s.d. `lag_7` | Nilai konsumsi listrik dari 1 hari lalu hingga 7 hari lalu. |
| **Input (Exogenous / $X$)**| `Temperature` | Rata-rata suhu udara pada hari tersebut (faktor eksternal). |

---

## 3. Apa itu *Lag*?
Dalam analisis deret waktu (*time series*), **Lag adalah nilai masa lalu dari variabel target itu sendiri** yang dipakai sebagai fitur masukan guna memprediksi masa depan. Karena data ini bersifat harian, maka artinya:
* **`lag_1`**: Konsumsi listrik 1 hari yang lalu (kemarin).
* **`lag_2`**: Konsumsi listrik 2 hari yang lalu.
* **`lag_7`**: Konsumsi listrik 7 hari yang lalu (hari yang sama di minggu lalu, berguna guna menangkap pola mingguan).

Model membutuhkan *lag* karena pola deret waktu hari ini umumnya memiliki keterkaitan atau ketergantungan yang kuat dengan apa yang terjadi pada hari-hari sebelumnya.

---

## 4. Proses Analisis yang Dilakukan
Eksperimen di dalam dokumentasi tersebut berjalan melalui 4 tahapan utama:

1. **Persiapan Data (*Data Preparation*):**
   Mengubah frekuensi data dari per 30 menit menjadi harian menggunakan `.resample('D')`. Konsumsi listrik dijumlahkan, sedangkan suhu dirata-rata. Data kemudian dibagi menjadi `data_train` dan `data_test`.
   
2. **Pelatihan Model (*Model Training*):**
   Membangun objek prediksi menggunakan `ForecasterRecursive` dengan algoritma **LightGBM** (`LGBMRegressor`). Model dikonfigurasi guna membaca 7 *lags* ke belakang serta menyertakan variabel eksogen `Temperature`.
   
3. **Evaluasi Tingkat Kepentingan Fitur (*Feature Importance*):**
   Memanfaatkan fungsi `forecaster.get_feature_importances()` guna melihat fitur yang paling dominan. Hasil visualisasi menunjukkan bahwa **Suhu (`Temperature`)** dan **`lag_1`** memiliki pengaruh terbesar bagi model dalam menentukan hasil prediksi.
   
4. **Visualisasi Keterjelasan Model (*SHAP Values*):**
   Memanfaatkan pustaka `shap` (khususnya `TreeExplainer`) guna membedah keputusan model secara visual:
   * **Summary Plot:** Melihat dampak global dan distribusi pengaruh dari setiap fitur.
   * **Force Plot:** Membedah alasan spesifik di balik satu poin prediksi tertentu (analisis lokal).
   * **Dependence Plot:** Melihat bagaimana grafik hubungan antara perubahan nilai suhu terhadap nilai prediksi yang dihasilkan.
