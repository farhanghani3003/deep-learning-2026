# Latihan Mandiri di Lab — Modul 05

**Nama:** Ahmad Farhan Ghani | **NIM:** 123450121 | **Kelas:** RA

NIM berakhiran **1** (rentang 0–4): kernel blok pertama diganti `Conv2d(3, 32, kernel_size=5, padding=2)`.

## Prediksi (hitungan tangan, masukan 3x32x32)

| Besaran | Baseline | Varian | Perhitungan |
|---|---|---|---|
| Parameter | 545 098 | **546 634** | blok 1: 32·3·25+32 = 2 432 (semula 896), naik 1 536 |
| Perkalian per citra | 6 128 896 | **7 701 760** | blok 1: 32·3·25·32² = 2 457 600 (semula 884 736), naik 1 572 864 |
| Receptive field keluaran blok 2 | 10 | **12** | 1+4=5; pool 6; conv 6+2·2=10; pool 10+1·2=12 |

Layer lain: p=2 mempertahankan ukuran 32x32, sehingga `Conv2d` kedua dan `Linear(4096, 128)` tidak berubah. Yang bergeser hanya receptive field layer-layer sesudah blok pertama.

## Hasil dan perbandingan

Sel latihan pada notebook memuat `assert aktual == PREDIKSI`, jadi angka `anatomi()` harus sama dengan tabel di atas. Salin keluaran tabel_varian dari notebook setelah dijalankan.