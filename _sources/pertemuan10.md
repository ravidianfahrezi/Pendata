# Laporan Proyek: Klasifikasi Naive Bayes Menggunakan KNIME dan Python (Sklearn)

## Pertemuan 10  
## Naive Bayes

---

## 1. Pengertian Naive Bayes

Naive Bayes adalah algoritma klasifikasi berbasis probabilitas yang menggunakan konsep dari Teorema Bayes. Algoritma ini disebut *Naive* atau naif karena mengasumsikan bahwa setiap fitur dalam data bersifat independen atau tidak saling memengaruhi satu sama lain.

Meskipun pada kenyataannya antar fitur dalam sebuah dataset sering kali saling berkaitan, asumsi sederhana ini membuat Naive Bayes tetap menjadi algoritma yang cepat, efisien, dan cukup baik untuk berbagai kasus klasifikasi.

---

## 2. Rumus Teorema Bayes

Rumus dasar Teorema Bayes adalah sebagai berikut:

```text
P(C|X) = (P(X|C) * P(C)) / P(X)
```

### Keterangan:

- **P(C|X)**: Peluang kelas C jika diketahui atribut X (*posterior*).
- **P(X|C)**: Peluang atribut X jika diketahui kelas C (*likelihood*).
- **P(C)**: Peluang awal munculnya kelas C (*prior*).
- **P(X)**: Probabilitas kemunculan atribut X (*evidence*).

---

## 3. Jenis-Jenis Naive Bayes

Beberapa jenis utama algoritma Naive Bayes adalah sebagai berikut:

1. **Gaussian Naive Bayes**  
   Digunakan untuk data dengan fitur numerik kontinu, seperti panjang dan lebar sepal atau petal pada dataset Iris.

2. **Multinomial Naive Bayes**  
   Umumnya digunakan untuk data teks, misalnya klasifikasi dokumen berdasarkan frekuensi kemunculan kata.

3. **Bernoulli Naive Bayes**  
   Digunakan untuk data biner, seperti data dengan nilai ya/tidak, benar/salah, atau 0/1.

---

## 4. Kelebihan Naive Bayes

Algoritma Naive Bayes memiliki beberapa kelebihan, yaitu:

- Proses pelatihan dan prediksi sangat cepat.
- Efisien secara komputasi.
- Cocok digunakan pada dataset berdimensi tinggi.
- Tidak membutuhkan data latih yang terlalu besar.
- Cukup baik digunakan untuk klasifikasi sederhana maupun menengah.

---

# TUGAS

## 5. Deskripsi Proyek

Proyek ini bertujuan untuk membangun model klasifikasi menggunakan algoritma **Gaussian Naive Bayes** dari library **scikit-learn** Python. Implementasi model dilakukan di dalam platform **KNIME Analytics Platform** dengan bantuan node **Python Script**.

Dataset yang digunakan adalah dataset **Iris**, yaitu dataset yang berisi data karakteristik bunga iris berdasarkan ukuran sepal dan petal. Dataset ini memiliki tiga kelas utama, yaitu:

- **Iris-setosa**
- **Iris-versicolor**
- **Iris-virginica**

Tujuan dari proyek ini adalah memprediksi jenis bunga iris berdasarkan fitur numerik yang tersedia dalam dataset.

---

## 6. Tools yang Digunakan

Tools dan library yang digunakan dalam proyek ini adalah:

- **KNIME Analytics Platform**  
  Digunakan untuk membuat alur kerja atau workflow secara visual.

- **Python Script Node**  
  Digunakan untuk menjalankan kode Python di dalam workflow KNIME.

- **Pandas**  
  Digunakan untuk mengolah data dalam bentuk DataFrame.

- **Scikit-learn**  
  Digunakan untuk membangun model machine learning menggunakan algoritma Gaussian Naive Bayes.

- **GaussianNB**  
  Model Naive Bayes dari library scikit-learn yang cocok untuk data numerik kontinu.

---

## 7. Langkah-Langkah Pembuatan Workflow

Berikut adalah tampilan keseluruhan workflow KNIME yang digunakan dalam proyek klasifikasi Naive Bayes.

![Gambar 1. Workflow utama KNIME](images/workflow_knime.png)

*Keterangan: Workflow terdiri dari node CSV Reader, Table Partitioner, Normalizer, Normalizer Apply, Python Script, Scorer, dan Table View.*

---

### 7.1 Membaca Data dengan CSV Reader

**Node:** CSV Reader

**Fungsi:**  
Node CSV Reader digunakan untuk mengimpor dataset Iris ke dalam environment KNIME.

**Konfigurasi:**  
Dataset dibaca dari direktori lokal komputer. File yang digunakan adalah file dataset Iris dalam format CSV. Pengaturan pembacaan data dapat disesuaikan dengan struktur file, seperti pemisah kolom, nama kolom, dan tipe data.

![Gambar 2. Konfigurasi CSV Reader](images/csv_reader.png)

*Keterangan: Node CSV Reader digunakan untuk membaca dataset Iris dari file CSV.*

---

### 7.2 Membagi Data Latih dan Data Uji dengan Table Partitioner

**Node:** Table Partitioner

**Fungsi:**  
Node Table Partitioner digunakan untuk membagi dataset menjadi dua bagian, yaitu:

- Data latih (*training data*)
- Data uji (*testing data*)

**Konfigurasi:**  
Pembagian data dilakukan menggunakan metode **Relative (%)** dengan rasio **70%** untuk data latih dan **30%** untuk data uji. Strategi sampling yang digunakan adalah **Random**, sehingga data dibagi secara acak.

Data dari port atas digunakan sebagai data latih, sedangkan data dari port bawah digunakan sebagai data uji.

![Gambar 3. Konfigurasi Table Partitioner](images/table_partitioner.png)

*Keterangan: Dataset dibagi menjadi 70% data latih dan 30% data uji dengan metode random sampling.*

---

### 7.3 Normalisasi Data Latih dengan Normalizer

**Node:** Normalizer

**Fungsi:**  
Node Normalizer digunakan untuk mengubah skala nilai fitur numerik agar berada dalam rentang yang seragam. Normalisasi penting dilakukan agar tidak ada fitur yang mendominasi proses klasifikasi hanya karena memiliki skala angka yang lebih besar.

**Konfigurasi:**  
Metode normalisasi yang digunakan adalah **Min-max normalization** dengan rentang nilai dari **0 hingga 1**.

Node ini menghasilkan data latih yang telah dinormalisasi serta model normalisasi yang menyimpan parameter minimum dan maksimum dari data latih.

![Gambar 4. Konfigurasi Normalizer](images/normalizer.png)

*Keterangan: Data latih dinormalisasi menggunakan metode Min-Max Normalization dengan rentang nilai 0 sampai 1.*

---

### 7.4 Normalisasi Data Uji dengan Normalizer Apply

**Node:** Normalizer (Apply)

**Fungsi:**  
Node Normalizer Apply digunakan untuk menerapkan model normalisasi dari data latih ke data uji.

**Konfigurasi:**  
Node ini menerima input model normalisasi dari node Normalizer melalui jalur model. Data uji kemudian dinormalisasi menggunakan parameter yang sama dengan data latih.

Langkah ini merupakan praktik yang baik dalam machine learning karena dapat mencegah terjadinya **data leakage**, yaitu kebocoran informasi dari data uji ke proses pelatihan model.

![Gambar 5. Konfigurasi Normalizer Apply](images/normalizer_apply.png)

*Keterangan: Model normalisasi dari data latih diterapkan pada data uji agar skala data tetap konsisten.*

---

### 7.5 Implementasi Naive Bayes dengan Python Script

**Node:** Python Script

**Fungsi:**  
Node Python Script digunakan untuk menjalankan kode Python yang berfungsi melatih model Gaussian Naive Bayes dan melakukan prediksi terhadap data uji.

**Proses yang dilakukan:**

1. Membaca data latih dan data uji dari input KNIME.
2. Memisahkan fitur dan target.
3. Melatih model Gaussian Naive Bayes menggunakan data latih.
4. Melakukan prediksi terhadap data uji.
5. Menambahkan hasil prediksi ke dalam kolom baru.
6. Mengirim hasil akhir kembali ke output KNIME.
7. Menampilkan laporan evaluasi menggunakan `classification_report`.

![Gambar 6. Node Python Script](images/python_script_node.png)

*Keterangan: Node Python Script digunakan untuk menjalankan kode Gaussian Naive Bayes di dalam workflow KNIME.*

![Gambar 7. Tampilan kode pada Python Script](images/python_script_code.png)

*Keterangan: Script Python digunakan untuk membaca data, melatih model, melakukan prediksi, dan mengirim hasil kembali ke KNIME.*

---

## 8. Script Python yang Digunakan

```python
import knime.scripting.io as knio
import pandas as pd
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import classification_report

# Membaca data latih dan data uji dari input KNIME
data_latih = knio.input_tables[0].to_pandas()
data_uji = knio.input_tables[1].to_pandas()

# Memisahkan fitur dan target pada data latih
fitur_latih = data_latih.iloc[:, :-1]
target_latih = data_latih.iloc[:, -1]

# Memisahkan fitur dan target pada data uji
fitur_uji = data_uji.iloc[:, :-1]
target_uji = data_uji.iloc[:, -1]

# Membuat model Gaussian Naive Bayes
model_nb = GaussianNB()

# Melatih model menggunakan data latih
model_nb.fit(fitur_latih, target_latih)

# Melakukan prediksi terhadap data uji
hasil_prediksi = model_nb.predict(fitur_uji)

# Menambahkan hasil prediksi ke dalam tabel data uji
hasil_akhir = data_uji.copy()
hasil_akhir["prediction"] = hasil_prediksi

# Mengirim hasil akhir kembali ke KNIME
knio.output_tables[0] = knio.Table.from_pandas(hasil_akhir)

# Menampilkan laporan evaluasi klasifikasi
print(classification_report(target_uji, hasil_prediksi))
```

---

## 9. Penjelasan Script

Pada script tersebut, data dari KNIME dibaca menggunakan `knio.input_tables`. Input pertama digunakan sebagai data latih, sedangkan input kedua digunakan sebagai data uji.

Selanjutnya, data dipisahkan menjadi fitur dan target. Fitur diambil dari semua kolom kecuali kolom terakhir, sedangkan target diambil dari kolom terakhir. Pada dataset Iris, kolom terakhir umumnya berisi label kelas atau spesies bunga.

Model yang digunakan adalah **GaussianNB**, yaitu implementasi Gaussian Naive Bayes dari library scikit-learn. Model dilatih menggunakan data latih, kemudian digunakan untuk memprediksi kelas dari data uji.

Hasil prediksi kemudian ditambahkan ke dalam data uji sebagai kolom baru bernama `prediction`. Data hasil prediksi tersebut dikirim kembali ke KNIME sebagai output tabel.

Selain itu, fungsi `classification_report` digunakan untuk menampilkan metrik evaluasi model seperti precision, recall, f1-score, dan support.

![Gambar 8. Output Classification Report](images/classification_report.png)

*Keterangan: Classification report menampilkan nilai precision, recall, f1-score, support, dan accuracy dari model.*

---

## 10. Evaluasi Model dengan Scorer

**Node:** Scorer

**Fungsi:**  
Node Scorer digunakan untuk membandingkan label asli dengan hasil prediksi model. Pada proyek ini, label asli adalah kolom **species**, sedangkan hasil prediksi adalah kolom **prediction**.

Metrik yang digunakan untuk mengevaluasi model antara lain:

- **Accuracy**
- **Precision**
- **Recall**
- **F1-score**
- **Confusion Matrix**
- **Cohen's Kappa**

![Gambar 9. Hasil evaluasi pada Node Scorer](images/scorer.png)

*Keterangan: Node Scorer digunakan untuk membandingkan kolom species sebagai label asli dengan kolom prediction sebagai hasil prediksi model.*

---

## 11. Hasil Evaluasi

Berdasarkan hasil evaluasi model, diperoleh nilai sebagai berikut:

- **Accuracy:** 0.956 atau 95,6%
- **Cohen's Kappa:** 0.933

### Confusion Matrix

| Kelas Aktual | Hasil Prediksi Benar | Kesalahan Prediksi |
|---|---:|---:|
| Iris-setosa | 16 | 0 |
| Iris-versicolor | 16 | 1 |
| Iris-virginica | 11 | 1 |

### Interpretasi Hasil

Model berhasil mengklasifikasikan kelas **Iris-setosa** dengan sangat baik tanpa kesalahan. Hal ini menunjukkan bahwa karakteristik Iris-setosa cukup berbeda dibandingkan dua kelas lainnya.

Kesalahan klasifikasi hanya terjadi pada kelas **Iris-versicolor** dan **Iris-virginica**. Hal tersebut wajar karena kedua kelas ini memiliki karakteristik ukuran sepal dan petal yang lebih mirip dibandingkan dengan Iris-setosa.

Secara umum, model menunjukkan performa yang sangat baik karena mampu mencapai akurasi sebesar **95,6%**.

---

## 12. Visualisasi Tabel Hasil dengan Table View

![Gambar 10. Tabel hasil prediksi pada Table View](images/table_view.png)

*Keterangan: Table View menampilkan data uji beserta kolom label asli dan kolom hasil prediksi.*

**Node:** Table View

**Fungsi:**  
Node Table View digunakan untuk menampilkan hasil akhir dari proses klasifikasi dalam bentuk tabel interaktif.

Tabel hasil berisi data uji yang telah dinormalisasi, label asli, serta kolom hasil prediksi. Dengan adanya kolom `prediction`, pengguna dapat melihat secara langsung apakah hasil prediksi model sesuai dengan label sebenarnya.

---

## 13. Kesimpulan

Berdasarkan proyek yang telah dilakukan, dapat disimpulkan bahwa algoritma **Gaussian Naive Bayes** dapat digunakan dengan baik untuk melakukan klasifikasi pada dataset Iris. Integrasi antara KNIME dan Python memudahkan proses penambangan data karena KNIME menyediakan workflow visual, sedangkan Python memberikan fleksibilitas dalam penggunaan library machine learning.

Model yang dibangun mampu mencapai akurasi sebesar **95,6%**, sehingga dapat dikatakan memiliki performa yang sangat baik. Kelas Iris-setosa berhasil diklasifikasikan dengan sempurna, sedangkan sedikit kesalahan terjadi pada kelas Iris-versicolor dan Iris-virginica karena kemiripan karakteristik antar keduanya.

Dengan demikian, proyek ini berhasil menunjukkan penerapan algoritma Naive Bayes dalam proses klasifikasi data menggunakan KNIME dan Python Sklearn.

---

## 14. Panduan Nama File Gambar

Agar gambar dapat muncul di file Markdown, simpan semua screenshot ke dalam folder bernama `images` yang berada satu lokasi dengan file laporan Markdown.

Struktur folder yang disarankan:

```text
laporan_naive_bayes_knime_python.md
images/
├── workflow_knime.png
├── csv_reader.png
├── table_partitioner.png
├── normalizer.png
├── normalizer_apply.png
├── python_script_node.png
├── python_script_code.png
├── classification_report.png
├── scorer.png
└── table_view.png
```

Jika nama file gambar berbeda, sesuaikan bagian alamat gambar di laporan. Contohnya:

```md
![Gambar 1. Workflow utama KNIME](images/workflow_knime.png)
```

Bagian `images/workflow_knime.png` dapat diganti sesuai nama file screenshot yang digunakan.

---

## 15. Daftar Pustaka

- Dokumentasi scikit-learn: Gaussian Naive Bayes.
- Dokumentasi KNIME Analytics Platform.
- Dataset Iris.
