# Template Matching dan Optical Flow untuk Pelacakan Objek pada Video Kamera Diam dan Kamera Bergerak

Penugasan Tutorial Computer Vision, Program Studi Magister Kecerdasan Artifisial, Universitas Gadjah Mada.

| | |
|---|---|
| Nama | Elsa Aiziyah |
| NIM | 25/572243/PPA/07187 |
| Notebook | https://drive.google.com/file/d/18wR-UtM7fuOpvCoIPaZTkhRygRrhyCbE/view?usp=sharing |
| Video hasil | `hasil_video1_kamera_diam.mp4` dan `hasil_video2_kamera_bergerak.mp4` |
| Laporan | `Laporan_IEEE_Tracking.pdf` (dan `.docx` yang dapat disunting) |

## Deskripsi singkat

Proyek ini mengimplementasikan dua algoritma pelacakan sederhana dan membandingkannya pada dua video:

1. **Template-based tracking.** Templat diambil dari kotak sasaran pada frame pertama, lalu dicari kembali di jendela pencarian sekitar posisi sebelumnya dengan `cv2.matchTemplate` (`TM_CCOEFF_NORMED`). Ukuran kotak dan templat tetap.
2. **Visual tracking by optical flow.** Titik sudut Shi–Tomasi di dalam kotak dilacak dengan Lucas–Kanade piramida, difilter dengan uji galat maju–mundur, lalu kotak digeser dan diskalakan dengan median aliran titik.

| Video | Sekuens | Kondisi kamera | Frame yang dipakai |
|---|---|---|---|
| V1 | FaceOcc2 (OTB-100) | diam | 812 frame, 32,5 detik |
| V2 | David (OTB-100) | bergerak (dipegang tangan) | 471 frame beranotasi, 18,8 detik |

Keduanya berukuran 320×240 piksel pada 25 FPS, bersasaran satu wajah, dan memiliki anotasi kotak pada setiap frame (*ground truth*). Kotak frame pertama dipakai untuk menginisialisasi kedua pelacak, sesuai protokol OTB.

## Hasil

Metrik mengikuti protokol OTB. Presisi@20 adalah proporsi frame dengan galat lokasi pusat (CLE) ≤ 20 piksel, AUC adalah luas di bawah kurva keberhasilan berbasis IoU, dan S@0,5 adalah proporsi frame dengan IoU > 0,5.

| Video | Pelacak | CLE rata-rata (px) | Presisi@20 | AUC | S@0,5 |
|---|---|---|---|---|---|
| V1 FaceOcc2 (diam) | Template | **10,4** | **77,6%** | **0,711** | **89,9%** |
| V1 FaceOcc2 (diam) | Aliran optik | 51,6 | 50,0% | 0,473 | 55,9% |
| V2 David (bergerak) | Template | 63,6 | 19,1% | 0,183 | 18,9% |
| V2 David (bergerak) | Aliran optik | **5,6** | **100,0%** | **0,638** | **86,0%** |

Temuan utama:

- **Kamera diam (V1).** Template matching tidak pernah kehilangan sasaran. Aliran optik sama baiknya selama wajah menghadap kamera, tetapi mulai frame 514 (detik 20,6) kotaknya berpindah ke buku yang menutupi wajah dan tidak kembali, karena pelacak ini tidak memiliki mekanisme menemukan kembali sasaran.
- **Kamera bergerak (V2).** Aliran optik tetap berada pada wajah sepanjang video. Template matching kehilangan sasaran mulai frame 103 (detik 4,1), karena penampilan wajah berubah akibat pencahayaan, ukuran, dan sudut pandang. Kotak aliran optik makin membesar relatif terhadap anotasi, sehingga IoU turun walaupun CLE tetap kecil.
- **Sensitivitas.** Pengaturan bawaan ditetapkan sebelum evaluasi dan tidak disetel pada kedua video. Uji sensitivitas dijalankan sesudahnya pada video yang sama sehingga bersifat eksploratif. Hasil aliran optik pada V1 sangat bergantung pada pengaturan (AUC 0,473–0,723), jadi kegagalan pada pengaturan bawaan menunjukkan kerapuhan, bukan batas pasti metodenya.

Analisis lengkap ada di notebook (bagian 7 sampai 11) dan di laporan.

## Menjalankan

**Google Colab (disarankan).** Buka notebook, lalu pilih *Runtime → Run all*. Data diunduh otomatis. Di mesin uji kami satu kali jalan memerlukan sekitar 35 detik. Di Colab waktunya bisa lebih lama.

**Lokal.** Python 3.10 atau lebih baru:

```bash
pip install numpy pandas matplotlib opencv-python jupyter
```

`ffmpeg` harus tersedia di PATH karena video hasil ditulis dengan encoder H.264. Jalankan notebook secara berurutan. Keluaran disimpan di `content_lokal/keluaran_tracking/` (atau `/content/keluaran_tracking/` di Colab) berupa dua video hasil, gambar (`fig_*.png`), `trajektori_per_frame.csv`, dan `ringkasan_metrik.json`.

## Reprodusibilitas

- Data diunduh dari repositori `opencv/opencv_extra` pada **commit tetap** `c9cdf09e4d4e5e73effeabce97ef52a5c2789afc` (direktori `testdata/cv/tracking`), lalu setiap berkas diverifikasi dengan SHA-256.
- Seed acak ditetapkan, dan fungsi metrik diuji pada kasus yang jawabannya dapat dihitung manual.
- Nilai kecepatan (FPS) bergantung perangkat. Metrik lain seharusnya sama, walaupun versi OpenCV yang berbeda dapat menghasilkan selisih kecil.

## Isi repositori

```
├── Tracking_Template_OpticalFlow.ipynb   Notebook (sudah dijalankan, beserta keluarannya)
├── hasil_video1_kamera_diam.mp4          Video hasil V1
├── hasil_video2_kamera_bergerak.mp4      Video hasil V2
├── Laporan_IEEE_Tracking.pdf             Laporan format IEEE
├── Laporan_IEEE_Tracking.docx            Laporan (dapat disunting)
└── README.md
```

Pada video hasil: kotak hijau adalah anotasi, biru adalah template matching, oranye adalah aliran optik, dan titik kuning adalah titik fitur aliran optik.

## Keterbatasan

- Hanya dua video dengan satu sasaran pada tiap video, sehingga perbedaan yang dilaporkan belum tentu berlaku umum.
- Pembagian segmen pada V1 ditentukan dari pengamatan frame.
- Kedua pelacak sengaja sederhana: tanpa penemuan kembali sasaran dan tanpa pembaruan templat pada pengaturan bawaan.

## Sumber

- Wu, Y., Lim, J., & Yang, M.-H. (2013). Online object tracking: A benchmark. *IEEE CVPR*, 2411–2418.
- Ross, D. A., Lim, J., Lin, R.-S., & Yang, M.-H. (2008). Incremental learning for robust visual tracking. *IJCV*, 77(1–3), 125–141. (David)
- Babenko, B., Yang, M.-H., & Belongie, S. (2011). Robust object tracking with online multiple instance learning. *IEEE TPAMI*, 33(8), 1619–1632. (FaceOcc2)
