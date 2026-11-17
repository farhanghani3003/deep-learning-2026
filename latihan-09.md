# Latihan Mandiri M09 - Jumlah Head dan Waktu Forward Pass

**Nama:** Ahmad Farhan Ghani | **NIM:** 123450121 | **Seed:** 121

## Prediksi (ditulis sebelum kode latihan dijalankan)

Parameter satu layer encoder sama untuk h ∈ {1, 2, 4, 10}, yaitu 76.075 (multi-head 40.400 + feed-forward 35.275 + dua LayerNorm 400), karena W_Q, W_K, W_V, W_O selalu berukuran d_model × d_model dan h hanya membagi kolomnya menjadi h potongan berukuran d_k = d_model / h.

Waktu forward pass (batch 64, n=60) diperkirakan naik sedikit seiring h (sekitar 0-15% dari h=1 ke h=10). Alasan yang dapat diuji: total perkalian matriks QK^T sama untuk semua h, tetapi tensor bobot attention berukuran B × h × n × n sehingga entri softmax tumbuh ∝ h. Jika benar, selisih h=10 terhadap h=1 harus lebih besar pada n=120 daripada n=60. Jika selisihnya di bawah rentang derau, prediksi tidak terbukti.

## Hasil

| h | d_k | Parameter multi-head | Parameter layer | ms (n=60, rentang) | ms (n=120, rentang) |
|---:|---:|---:|---:|---:|---:|
| 1 | 100 | 40.400 | 76.075 | 10,35 (3,24) | 21,33 (1,39) |
| 2 | 50 | 40.400 | 76.075 | 11,06 (1,36) | 30,37 (2,61) |
| 4 | 25 | 40.400 | 76.075 | 13,33 (0,73) | 64,29 (10,87) |
| 10 | 10 | 40.400 | 76.075 | 25,25 (2,38) | 111,04 (4,98) |

Rasio waktu h=10 / h=1: 2,44 (n=60) dan 5,21 (n=120).

## Perbandingan dengan prediksi

- Parameter: prediksi terbukti (76.075 untuk semua h; cocok dengan PyTorch).
- Arah waktu: naik seiring h, sesuai prediksi.
- Besar waktu: meleset jauh. Prediksi 0-15%, hasilnya h=10 adalah 2,44 kali h=1 (+144%) pada n=60. Selisih h=1 dan h=2 (0,71 ms) masih di dalam rentang derau (3,24 ms), tetapi selisih h=1 dan h=10 (14,9 ms) jauh di atasnya.
- Alasan yang diuji: selisih pada n=120 (5,21 kali) lebih besar daripada pada n=60 (2,44 kali), konsisten dengan entri softmax ∝ h n². Uji ini tidak memisahkan mekanismenya: d_k yang kecil (10) membuat perkalian matriks per head kurang efisien di CPU, dan itu juga dapat menjelaskan kenaikan tersebut.
