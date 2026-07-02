# Pertemuan 14

# Analisis Performa Akademik Mahasiswa Menggunakan Metode Klasifikasi

## Latar Belakang

Perkembangan teknologi informasi memungkinkan institusi pendidikan memanfaatkan data akademik mahasiswa sebagai dasar dalam proses pengambilan keputusan. Salah satu pemanfaatannya adalah melakukan analisis terhadap data performa mahasiswa untuk mengetahui pola yang dapat digunakan dalam memprediksi kategori nilai akhir mahasiswa.

Prediksi nilai akhir mahasiswa dapat membantu institusi pendidikan dalam memahami karakteristik mahasiswa serta menjadi dasar dalam melakukan evaluasi proses pembelajaran. Untuk memperoleh model prediksi yang baik, diperlukan pengujian beberapa algoritma klasifikasi sehingga dapat diketahui metode yang paling sesuai terhadap dataset yang digunakan.

Pada analisis ini digunakan dataset **Higher Education Students Performance Evaluation** yang berisi data karakteristik pembelajaran mahasiswa beserta kategori nilai akhirnya (**GRADE**). Proses analisis dilakukan menggunakan aplikasi **Orange Data Mining** dengan membandingkan empat algoritma klasifikasi, yaitu **Decision Tree**, **Random Forest**, **k-Nearest Neighbor (kNN)**, dan **Naive Bayes**.

---

# 1. Business Understanding

Tujuan utama analisis ini adalah membangun model klasifikasi yang mampu memprediksi kategori nilai akhir (**GRADE**) mahasiswa berdasarkan data karakteristik dan aktivitas pembelajaran mahasiswa.

Selain melakukan prediksi, penelitian ini juga bertujuan membandingkan beberapa algoritma klasifikasi untuk mengetahui metode yang memberikan performa terbaik pada dataset yang digunakan.

Karena target yang diprediksi berupa kategori nilai (**GRADE**), maka permasalahan ini termasuk ke dalam **klasifikasi (classification)**.

---

# 2. Data Understanding

Dataset yang digunakan adalah **Higher Education Students Performance Evaluation**.

Karakteristik dataset:

- Jumlah data: **145 mahasiswa**
- Jumlah atribut: **33 kolom**

Struktur dataset terdiri dari:

| Atribut | Keterangan |
|----------|------------|
| STUDENT ID | Identitas mahasiswa |
| COURSE ID | Kode mata kuliah |
| 1 - 30 | Atribut yang menggambarkan karakteristik dan aktivitas pembelajaran mahasiswa |
| GRADE | Kategori nilai akhir mahasiswa (Target) |

Pada tahap ini dilakukan pemahaman terhadap struktur dataset untuk mengetahui atribut yang akan digunakan dalam proses klasifikasi.

---

# 3. Preprocessing Data

Tahap preprocessing dilakukan untuk menyiapkan data sebelum proses pemodelan.

## 3.1 Import Dataset

Dataset diimpor ke dalam Orange menggunakan widget **File**, kemudian dilakukan pengecekan isi data menggunakan widget **Data Table** untuk memastikan seluruh data berhasil dimuat dengan benar.

---

## 3.2 Seleksi Atribut

Proses seleksi atribut dilakukan menggunakan widget **Select Columns**.

Pengaturan atribut sebagai berikut:

### Feature

- COURSE ID
- Kolom 1 sampai 30

### Target

- GRADE

### Meta

- STUDENT ID

Pengaturan ini bertujuan agar model hanya menggunakan atribut yang relevan untuk melakukan proses klasifikasi, sedangkan STUDENT ID hanya digunakan sebagai identitas data.

---

# 4. Modeling

Pada tahap pemodelan digunakan empat algoritma klasifikasi.

## 4.1 Decision Tree

Decision Tree membangun model klasifikasi dalam bentuk pohon keputusan sehingga proses pengambilan keputusan dapat dipahami melalui aturan-aturan yang terbentuk.

---

## 4.2 Random Forest

Random Forest merupakan pengembangan dari Decision Tree yang membangun banyak pohon keputusan, kemudian menggabungkan hasil prediksi dari seluruh pohon untuk menghasilkan model yang lebih stabil.

---

## 4.3 k-Nearest Neighbor (kNN)

Algoritma kNN melakukan klasifikasi berdasarkan kemiripan data. Suatu data baru akan dikelompokkan ke kelas yang paling banyak dimiliki oleh sejumlah tetangga terdekatnya.

---

## 4.4 Naive Bayes

Naive Bayes menggunakan pendekatan probabilitas dalam melakukan klasifikasi dengan asumsi bahwa setiap atribut bersifat independen terhadap atribut lainnya.

---

# 5. Evaluasi Model

Evaluasi dilakukan menggunakan widget **Test & Score** pada Orange dengan metode **10-Fold Cross Validation**.

Metode ini digunakan agar setiap data memperoleh kesempatan menjadi data latih maupun data uji sehingga hasil evaluasi menjadi lebih objektif.

Hasil pengujian ditunjukkan pada tabel berikut.

| Algoritma | AUC | Accuracy (CA) | F1 Score | Precision | Recall |
|-----------|----:|--------------:|---------:|----------:|-------:|
| Decision Tree | 0.602 | 27.6% | 0.270 | 0.268 | 0.276 |
| Random Forest | 0.643 | 26.9% | 0.262 | 0.278 | 0.269 |
| k-Nearest Neighbor | 0.638 | **28.3%** | 0.264 | 0.268 | 0.283 |
| Naive Bayes | **0.695** | 24.1% | 0.221 | **0.297** | 0.241 |

### Analisis Hasil

Berdasarkan nilai **Classification Accuracy (CA)**, algoritma **k-Nearest Neighbor (kNN)** memperoleh nilai tertinggi sebesar **28,3%**, sehingga dipilih sebagai algoritma dengan performa terbaik pada dataset ini.

Walaupun algoritma **Naive Bayes** memiliki nilai **AUC** dan **Precision** yang lebih tinggi, pada penelitian ini indikator utama yang digunakan dalam menentukan model terbaik adalah **Classification Accuracy**, karena metrik tersebut menunjukkan persentase prediksi yang benar dari seluruh data yang diuji.

---

# 6. Analisis Confusion Matrix

Setelah proses evaluasi selesai, dilakukan analisis menggunakan **Confusion Matrix**.

Confusion Matrix digunakan untuk mengetahui jumlah prediksi yang benar maupun prediksi yang salah pada setiap kategori **GRADE**.

Nilai yang berada pada diagonal utama menunjukkan jumlah data yang berhasil diprediksi dengan benar, sedangkan nilai di luar diagonal menunjukkan data yang mengalami kesalahan klasifikasi.

Berdasarkan hasil Confusion Matrix, masih terdapat cukup banyak prediksi yang berada di luar diagonal utama. Hal ini menunjukkan bahwa model masih mengalami kesalahan dalam membedakan beberapa kategori GRADE mahasiswa.

Hasil tersebut sejalan dengan nilai Accuracy yang masih berada pada kisaran **24% hingga 28%**, sehingga kemampuan model dalam melakukan klasifikasi masih belum optimal.

---

# 7. Kesimpulan

Berdasarkan hasil analisis terhadap dataset **Higher Education Students Performance Evaluation**, dapat disimpulkan bahwa permasalahan yang dianalisis termasuk ke dalam kasus **klasifikasi**, karena bertujuan memprediksi kategori nilai akhir (**GRADE**) mahasiswa berdasarkan karakteristik pembelajaran mahasiswa.

Empat algoritma klasifikasi telah diuji, yaitu **Decision Tree**, **Random Forest**, **k-Nearest Neighbor (kNN)**, dan **Naive Bayes**.

Hasil evaluasi menunjukkan bahwa algoritma **k-Nearest Neighbor (kNN)** memperoleh nilai **Classification Accuracy (CA)** tertinggi sebesar **28,3%**, sehingga menjadi algoritma dengan performa terbaik pada dataset yang digunakan.

Meskipun demikian, performa model secara keseluruhan masih tergolong rendah. Hal ini terlihat dari nilai Accuracy yang masih berada di bawah 30% serta hasil Confusion Matrix yang menunjukkan masih banyak kesalahan klasifikasi pada beberapa kategori GRADE.

Kondisi tersebut menunjukkan bahwa model masih mengalami kesulitan dalam mengenali pola setiap kategori nilai mahasiswa. Salah satu penyebabnya adalah jumlah data yang relatif sedikit serta distribusi data pada setiap kategori GRADE yang tidak seimbang, sehingga proses pembelajaran model belum dapat menghasilkan prediksi yang optimal.