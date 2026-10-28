### Hasil dan perbandingan dengan prediksi

**Jawaban:**

Hasil tiga run (baseline, dropout 0,2, dropout 0,5). Kolom "terbaik" diukur pada epoch dengan validation loss terendah, kolom "terakhir" pada epoch ke-30.

| Run | Epoch terbaik | Val loss terbaik | Train loss terakhir | Val loss terakhir | Gap terakhir | Val acc |
|:---|---:|---:|---:|---:|---:|---:|
| baseline | 6 | 0,4565 | 0,0604 | 0,8371 | 0,7767 | 0,8357 |
| dropout_0.2 | 10 | 0,4678 | 0,0553 | 0,6546 | 0,5993 | 0,8323 |
| dropout_0.5 | 17 | 0,4524 | 0,1423 | 0,5237 | 0,3815 | 0,8417 |

**Perbandingan dengan prediksi:**

- **Gap epoch terakhir: sesuai.** Gap mengecil berurutan, 0,7767 > 0,5993 > 0,3815.
- **Train loss: sebagian sesuai.** Dropout 0,5 memang paling tinggi (0,1423, sekitar 2,4 kali baseline), tetapi dropout 0,2 (0,0553) sedikit di bawah baseline (0,0604), bukan di atasnya. Selisih 0,005 itu kecil dan berasal dari satu seed, jadi belum bisa disebut efek nyata.
- **Validation loss dropout 0,2: tidak sesuai.** Prediksi saya lebih rendah atau hampir sama dengan baseline, hasilnya 0,4678 vs 0,4565 (lebih tinggi 0,0113).
- **Validation loss dropout 0,5: tidak sesuai.** Saya memperkirakan tidak jauh lebih baik dari dropout 0,2, tetapi 0,4524 adalah yang terbaik dari ketiga run (lebih rendah 0,0154 dari dropout 0,2).

Dropout juga menunda overfitting: epoch terbaik bergeser dari 6 ke 10 ke 17, dan kenaikan validation loss setelah epoch terbaik makin landai (val loss epoch terakhir 0,8371 → 0,6546 → 0,5237). Selisih validation loss terbaik antar run kecil (0,4524 sampai 0,4678) dan hanya satu seed per konfigurasi, sehingga tidak cukup untuk menyimpulkan nilai dropout optimal.

**Mengapa dropout yang terlalu besar dapat memperburuk kedua angka sekaligus:**

1. **Kapasitas efektif turun.** Dropout p mematikan sebagian p unit pada tiap langkah training. Pada p = 0,5 dengan dua lapisan tersembunyi, hanya sekitar separuh unit aktif per lapisan, sehingga sinyal ke lapisan keluaran dan gradiennya berisik.
2. **Underfitting.** Jika p terlalu besar, model belum menangkap pola data. Train loss tetap tinggi, dan karena pola belum dipelajari, validation loss ikut tinggi. Keduanya memburuk bersama.
3. **Gap kecil bukan bukti model bagus.** Gap bisa mengecil karena train loss naik, bukan karena validation loss turun, sehingga nilai dropout harus dipilih dari validation loss.
4. **Data kecil (3.000 citra)** membuat jumlah pembaruan per epoch sedikit, sehingga model yang diberi noise besar lebih lambat mencapai loss rendah dalam 30 epoch.

Pada percobaan ini tanda awalnya terlihat di dropout 0,5 (train loss epoch terakhir naik menjadi 0,1423), tetapi validation loss masih membaik, jadi 0,5 masih di sisi yang menguntungkan. Dropout yang lebih besar (misalnya 0,7) tidak diuji, sehingga poin 1 sampai 4 adalah penjelasan konsep, bukan hasil percobaan.