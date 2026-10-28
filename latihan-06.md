# Catatan Latihan 06 — Augmentasi Agresif vs Ringan

| Komponen | Isian |
|---|---|
| Nama | Ginda Fajar Riadi Marpaung |
| NIM | 123450103 |
| Seed | 103 (empat digit terakhir NIM) |
| Lingkungan | PyTorch 2.11.0+cu130 |
| Dataset | Subset 4 kelas turunan Fashion-MNIST (`data/raw/transfer`, 120 citra per kelas) |

## 1. Prediksi (ditulis sebelum menjalankan kode)

Validation accuracy augmentasi agresif akan sama atau sedikit lebih rendah dibanding augmentasi ringan (kira-kira turun 0–5 poin). Alasannya, crop skala 0,3 dan rotasi ±45° membuat data latih jauh lebih sulit, sementara anggaran hanya 5 epoch (55 update), sehingga model cenderung underfit. Saya perkirakan train loss lebih tinggi dan gap lebih kecil daripada augmentasi ringan.

## 2. Hasil aktual

Strategi `partial` (layer4 + head), lr 1e-4, batch 32, 5 epoch (55 update), seed 103. Augmentasi agresif: `RandomResizedCrop(224, scale=(0,3; 1,0))`, `RandomRotation(45)`, `RandomHorizontalFlip`, lalu `ToTensor` dan `Normalize` dengan mean dan std dari `weights.transforms()`. Augmentasi hanya dikenakan pada split latih. Validation set berisi 72 citra, sehingga 1 citra = 0,014 accuracy.

| Run | Augmentasi | Train loss | Train acc | Val loss | Val acc | Gap |
|---|---|---:|---:|---:|---:|---:|
| partial-layer4 | ringan | 0,1298 | 0,988 | 0,6161 | 0,736 (53/72) | 0,4863 |
| partial-tanpa-augmentasi | tanpa | 0,0213 | 1,000 | 0,6426 | 0,708 (51/72) | 0,6213 |
| partial-augmentasi-agresif | agresif | 0,5979 | 0,762 | 0,7808 | 0,681 (49/72) | 0,1829 |

Train loss dan train acc dihitung ulang pada citra latih tanpa augmentasi, dengan cara yang sama seperti validation.

## 3. Perbandingan prediksi dengan hasil

Arah prediksi sesuai. Dari augmentasi ringan ke agresif, validation accuracy turun 0,056 (4 dari 72 citra), validation loss naik dari 0,616 menjadi 0,781, train loss naik dari 0,130 menjadi 0,598, dan gap menyempit dari 0,486 menjadi 0,183. Besar penurunannya sedikit melampaui rentang prediksi saya (0–5 poin), yaitu sekitar 5,6 poin.

Gap yang kecil pada run agresif bukan tanda generalisasi yang lebih baik. Train accuracy-nya hanya 0,762 pada citra latih tanpa augmentasi, sehingga model belum cocok bahkan dengan data latihnya sendiri: ini underfitting pada anggaran 55 update. Pada grafik validation accuracy, run agresif mulai sekitar 0,43 dan masih naik di epoch 5, sedangkan run ringan sudah mencapai sekitar 0,79 pada epoch 3. Saya menduga run agresif membutuhkan lebih dari 5 epoch, tetapi dugaan ini tidak diuji dalam protokol ini.

Validation loss, train loss, dan seluruh kurva epoch bergerak searah, jadi penurunan ini bukan sekadar selisih satu atau dua citra. Namun besar selisih yang pasti tidak dapat disimpulkan dari satu seed dan 72 citra validasi. Perbedaan ringan terhadap tanpa augmentasi hanya 2 citra (0,736 vs 0,708), sehingga tidak cukup untuk menyatakan ringan lebih baik secara tegas.

## 4. Kelas yang cirinya dapat terhapus oleh augmentasi agresif

Pengamatan berikut berasal dari visualisasi empat variasi agresif per kelas. Akurasi per kelas tidak diukur, jadi ini hipotesis dari gambar dan belum diuji.

Augmentasi agresif paling berbahaya bagi kelas yang ciri penentunya kecil atau berada di tepi citra, dan bagi kelas yang tampak polos setelah diperbesar:

- **pullover**: crop berskala kecil memperbesar badan hingga hampir hanya blok putih polos. Leher, manset lengan, dan ujung bawah, yang menjadi ciri kelasnya, hilang.
- **t_shirt_top**: pada sebagian variasi lengan pendek terpotong keluar bingkai sehingga bentuknya mirip badan tanpa lengan, padahal panjang lengan adalah pembeda utamanya dari `shirt`.
- **shirt**: rotasi hingga 45° menyilangkan siluet badan dan lengan, sedangkan kerah dan placket sering terpotong di pojok.
- **coat** relatif aman, karena deretan kancing di tengah besar dan tetap terlihat pada semua variasi.

Kerusakan ciri ini lebih serius karena citra asli hanya 28×28 yang diperbesar ke 224×224, sehingga detail halus sudah minim sebelum augmentasi.
