# Pertemuan 5

Data mentah dalam Data Mining sering kali memiliki skala atribut yang sangat timpang karena awalnya dirancang untuk kebutuhan operasional, bukan analisis. Ketimpangan ini dapat mendistorsi hasil algoritma yang berbasis perhitungan jarak, di mana atribut bernilai besar akan mendominasi atribut kecil secara tidak proporsional. Untuk mengatasinya, diperlukan teknik Normalisasi Data—seperti Min-Max Normalization, Z-Score, atau Decimal Scaling—yang mentransformasi nilai ke dalam skala seragam tanpa menghilangkan informasi esensial, sehingga memastikan setiap atribut memberikan kontribusi yang adil dalam proses pemodelan.

# Z-Score Normalization

Z-Score Normalization atau sering disebut standardisasi merupakan metode normalisasi yang mengubah distribusi data sehingga memiliki nilai rata-rata sebesar 0 dan standar deviasi sebesar 1. Dengan cara ini, data akan memiliki distribusi yang lebih seimbang sehingga memudahkan proses analisis.

Metode ini biasanya digunakan ketika nilai minimum atau maksimum suatu atribut tidak diketahui secara pasti, atau ketika dataset mengandung outlier yang dapat memengaruhi skala data jika menggunakan metode normalisasi lain seperti Min-Max.

**Rumus Z-Score**
$v' = \frac{v - \bar{A}}{\sigma_A}$

*Keterangan*
- v = nilai asli data
- v' = nilai hasil normalisasi
- Ā = rata-rata dari atribut A
- σA = standar deviasi atribut A

Sebagai contoh, misalkan terdapat data: `[10, 20, 30, 40, 100]`, Nilai 100 dapat dianggap sebagai outlier.

*Hasil perhitungan:*
- Rata-rata = 40
- Standar deviasi ≈ 31.62

*Hasil Normalisasi*
- 10 => -0.94
- 40 => 0.00
- 100 => 1.89
Nilai 0 menunjukkan bahwa data berada tepat pada nilai rata-rata.

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

# membuat dataset contoh
data = {'nilai': [10, 20, 30, 40, 100]}
df = pd.DataFrame(data)

# membuat objek scaler
scaler = StandardScaler()

# melakukan normalisasi Z-score
df['zscore_sklearn'] = scaler.fit_transform(df[['nilai']])

# menampilkan hasil
print(df)
```

![](images/pertemuan5/zscore-hasil.png)


# Min-Max Normalization

**Min-Max** Normalization merupakan metode normalisasi yang bertujuan untuk mengubah nilai atribut numerik ke dalam rentang tertentu, biasanya 0 sampai 1. Teknik ini sering digunakan agar semua atribut memiliki skala yang sebanding.

Metode ini banyak digunakan pada algoritma yang berbasis perhitungan jarak, seperti K-Nearest Neighbors, karena tanpa normalisasi atribut dengan nilai yang lebih besar dapat mendominasi hasil perhitungan.

**Rumus Min-Max Normalization**
$v' = \frac{v - min_A}{max_A - min_A} (new\_max_A - new\_min_A) + new\_min_A$

Jika rentang yang digunakan adalah 0 sampai 1, maka rumusnya dapat disederhanakan menjadi:
$v' = \frac{v - min_A}{max_A - min_A}$

*Keterangan*
- v = nilai asli data
- v' = nilai setelah normalisasi
- minA = nilai minimum pada atribut A
- maxA = nilai maksimum pada atribut A
- new_minA = batas bawah rentang baru
- new_maxA = batas atas rentang baru

Contoh data: `[10, 20, 30, 40, 100]` 
- Nilai minimum = 10
- Nilai maksimum = 100

*Hasil Normalisasi*
- 10 => -0.0
- 40 => 0.33
- 100 => 1.0

```python
import pandas as pd
from sklearn.preprocessing import MinMaxScaler

# membuat dataset contoh
data = {'nilai': [55, 60, 65, 70, 95]}
df = pd.DataFrame(data)

# membuat objek scaler
scaler = MinMaxScaler()

# melakukan normalisasi
df['minmax'] = scaler.fit_transform(df[['nilai']])

# membulatkan 3 angka di belakang koma
df['minmax'] = df['minmax'].round(3)

# menampilkan hasil
print(df)
```
![](images/pertemuan5/hasil-minmax.png)


# Decimal Scaling

Decimal Scaling merupakan metode normalisasi yang dilakukan dengan cara menggeser posisi titik desimal pada nilai data. Proses ini dilakukan dengan membagi nilai atribut dengan pangkat sepuluh tertentu.

Tujuan dari metode ini adalah agar nilai absolut maksimum dari atribut menjadi lebih kecil dari 1.

**Rumus Decimal Scaling**
$v' = \frac{v}{10^j}$

Keterangan
- v = nilai asli data
- v' = nilai setelah normalisasi
- j = bilangan bulat terkecil yang membuat nilai maksimum atribut menjadi kurang dari 1
Nilai j biasanya ditentukan dari jumlah digit angka terbesar dalam atribut.

Contoh data: `[10, 20, 30, 40, 100]`

Nilai terbesar = 100
Jumlah digit = 3

Sehingga semua nilai dibagi dengan 10³ (1000).

Hasil normalisasi:
- 10 => 0.01
- 40 => 0.04
- 100 => 0.10

```python
import pandas as pd
import numpy as np

# dataset contoh
data = {'nilai': [10, 20, 30, 40, 100]}
df = pd.DataFrame(data)

# mencari nilai maksimum
max_val = df['nilai'].abs().max()

# menentukan nilai j
j = len(str(int(max_val)))

# melakukan decimal scaling
df['decimal_scaling'] = df['nilai'] / (10**j)

# membulatkan hasil
df['decimal_scaling'] = df['decimal_scaling'].round(3)

print(df)
```
![](images/pertemuan5/hasil-minmax.png)