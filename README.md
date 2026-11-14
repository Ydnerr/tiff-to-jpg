# Convert TIF: TIFF jadi JPG/PNG di GitHub Actions

Mengubah file TIFF (8-bit atau 16-bit) jadi JPG atau PNG yang bisa dibuka di galeri HP, tanpa mengubah resolusi. Cocok untuk hasil stacking (`<nama>.tif`) yang tidak bisa dibuka di galeri Android.

## 1. Setup (sekali saja)

Simpan `convert-tif.yml` di repo sebagai:

```
.github/workflows/convert-tif.yml
```

Dari browser HP: **Add file → Create new file**, ketik nama `.github/workflows/convert-tif.yml` (garis miring otomatis membuat folder), tempel isinya, lalu **Commit changes**.

Tidak ada file lain yang perlu diupload. Setelah itu, di tab **Actions** muncul workflow baru bernama **Convert TIF**.

## 2. Pakai: TIFF dari hasil run stacking sebelumnya

Ini cara yang paling sering dipakai, karena hasil stack `.tif` ada di **Artifacts**, bukan di repo.

**Langkah A. Ambil run_id**
1. Buka **Actions**, lalu buka run stacking yang hasilnya mau dikonversi (yang hijau).
2. Lihat URL di browser. Bentuknya `github.com/<user>/<repo>/actions/runs/`**`12345678901`**. Angka di akhir itu `run_id`.

**Langkah B. Jalankan konversi**
1. **Actions → Convert TIF → Run workflow**.
2. Isi form:

| Kolom | Isi |
| --- | --- |
| sumber | `artifact-run-sebelumnya` |
| path | kosongkan |
| run_id | angka dari langkah A |
| filter | nama file yang mau diambil, mis. `contoh` (kosongkan untuk semua TIFF di artifact) |
| kualitas | `jpg-100` |
| commit_back | centang |

3. Tekan **Run workflow**, tunggu sampai hijau.

Artifact dari run stacking disimpan 30 hari. Setelah itu `run_id` tidak bisa dipakai lagi.

Kalau run sebelumnya adalah `stack-lalu-grade`, artifact-nya berisi banyak TIFF (`<nama>.tif`, `..._graded.tif`, `..._starreduce.tif`, `..._starreduce2.tif`, dan lainnya). Isi **filter** dengan potongan nama yang kamu mau, mis. `starreduce2`, supaya yang dikonversi cuma itu.

## 3. Pakai: TIFF yang ada di repo

1. **Actions → Convert TIF → Run workflow**.
2. Isi form:

| Kolom | Isi |
| --- | --- |
| sumber | `file-di-repo` |
| path | file `.tif`, mis. `sesi1/DSC-0042.tif`, atau folder, mis. `sesi1` |
| run_id | kosongkan |
| filter | opsional |
| kualitas | `jpg-100` |

Kalau path berupa folder, yang dikonversi hanya file `.tif` dan `.tiff` yang ada langsung di folder itu (subfolder tidak ikut).

## 4. Hasilnya di mana

- Kalau **commit_back** dicentang: di folder **`hasil-jpg/`** di repo. Buka lewat tab **Code** (atau app GitHub), lalu tap filenya. Gambar langsung tampil.
- Selalu ada juga di **Artifacts** pada halaman run (`hasil-jpg-...`). File di Artifacts berbentuk zip: unduh, lalu ekstrak dulu di file manager sebelum dibuka.
- Halaman **Summary** pada run menampilkan tabel ukuran, tipe data, dan nilai kemiripan tiap file.

Nama hasil sama dengan nama file asal (`DSC-0042.tif` jadi `DSC-0042.jpg`). Kalau ada dua file bernama sama, yang kedua diberi akhiran `_2`.

## 5. Pilihan kualitas

| Pilihan | Artinya | Kapan dipakai |
| --- | --- | --- |
| `jpg-100` | JPG kualitas tertinggi, warna penuh (4:4:4, tanpa pengecilan warna) | Default. Mendekati 1:1 dan bisa dibuka di mana saja. |
| `jpg-95` | JPG kualitas 95, warna penuh | Kalau file `jpg-100` terlalu besar. |
| `png-lossless` | PNG 8-bit tanpa kehilangan | Kalau mau persis sama dengan versi 8-bit-nya. File paling besar. |

Nilai **kemiripan** di Summary adalah PSNR antara gambar 8-bit sebelum dan sesudah disimpan. Di atas 45 dB, perbedaannya tidak terlihat mata. Pada uji coba gue, `jpg-100` ada di kisaran 50 sampai 58 dB dan `jpg-95` sekitar 45 sampai 47 dB. `png-lossless` ditulis sebagai "identik".

## 6. Apa yang terjadi pada gambar

- **Resolusi tidak diubah.** Lebar dan tinggi hasil sama dengan TIFF asal.
- **16-bit jadi 8-bit** dengan pembulatan yang benar (65535 jadi 255). JPG dan PNG di sini hanya 8-bit, jadi presisi 16-bit hilang. Simpan TIFF aslinya kalau masih mau diolah lagi.
- Profil warna (ICC) disalin kalau ada di TIFF. Kalau tidak ada, gambar dianggap sRGB.
- Hanya halaman pertama TIFF yang diambil. Kanal alpha (transparansi) dibuang.
- Tipe yang didukung: 8-bit, 16-bit, dan float (0 sampai 1), baik berwarna maupun abu-abu.

## 7. Kalau gagal

Buka run yang merah, lalu buka langkah yang merah dan cari baris `Error:`.

| Pesan | Penyebab dan perbaikan |
| --- | --- |
| `run_id harus angka saja` | Isi kolom run_id dengan angka dari URL, tanpa teks lain. |
| `Isi kolom path` | sumber = `file-di-repo` tapi path kosong. |
| `path tidak valid` | Path memakai karakter yang tidak diizinkan, diawali `/`, atau mengandung `..`. |
| `'...' tidak ada di repo` | Path salah ketik atau salah huruf besar-kecil. |
| `Tidak ada file .tif/.tiff yang cocok` | Folder tidak berisi TIFF, filter terlalu spesifik, atau artifact tidak berisi TIFF. |
| `Unable to download artifact(s)` atau `Artifact not found` | run_id salah, run itu bukan dari repo ini, atau artifact sudah lewat 30 hari. |
| `tipe data ... belum didukung` | TIFF memakai tipe selain 8-bit, 16-bit, atau float. File lain tetap dikonversi, tapi run ditandai gagal. |
| `git push` gagal (403) | **Settings → Actions → General → Workflow permissions**, pilih **Read and write**. Atau matikan commit_back dan ambil dari Artifacts. |

Satu file hasil yang lebih dari 100 MB tidak bisa di-commit ke repo. Matikan **commit_back** untuk file sebesar itu dan ambil dari Artifacts.

## 8. Catatan

- `hasil-jpg/` ikut tersimpan di riwayat git. Kalau sering konversi file besar, repo bisa membengkak. Hapus isi folder itu dari tab **Code** kalau sudah tidak perlu.
- `stack.py` sudah otomatis membuat `<nama>_preview.jpg` (8-bit, kualitas 92) saat stacking. Workflow ini untuk kalau mau versi yang lebih dekat ke TIFF aslinya.
