# Hermes Agent di Android (Termux + proot Ubuntu)

Panduan langkah demi langkah untuk pemula: memasang dan menjalankan **Hermes Agent** (agen AI dari Nous Research) di HP Android, tanpa root, dengan opsi model **gratis**.

> **Status panduan:** diuji 7 Oktober 2026 pada Android arm64 (aarch64), Termux dari F-Droid, Ubuntu lewat `proot-distro`, Python 3.14, Hermes v0.21.5.
> Panduan ini **tidak resmi**. Hermes adalah proyek Nous Research (lisensi MIT). Selalu cocokkan dengan [dokumentasi resmi](https://hermes-agent.nousresearch.com/docs/) karena versi baru bisa mengubah langkah.

---

## Daftar Isi

1. [Baca dulu: kenapa lewat proot Ubuntu?](#1-baca-dulu-kenapa-lewat-proot-ubuntu)
2. [Alur singkat](#2-alur-singkat)
3. [Langkah 1: Pasang Termux](#3-langkah-1-pasang-termux)
4. [Langkah 2: Pasang Ubuntu (proot-distro)](#4-langkah-2-pasang-ubuntu-proot-distro)
5. [Langkah 3: Siapkan Ubuntu](#5-langkah-3-siapkan-ubuntu)
6. [Langkah 4: Pasang Python 3.14](#6-langkah-4-pasang-python-314)
7. [Langkah 5: Ambil kode Hermes](#7-langkah-5-ambil-kode-hermes)
8. [Langkah 6: Buat venv dan pasang Hermes](#8-langkah-6-buat-venv-dan-pasang-hermes)
9. [Langkah 7: Pilih provider model](#9-langkah-7-pilih-provider-model)
10. [Langkah 8: Jalankan Hermes](#10-langkah-8-jalankan-hermes)
11. [Rutinitas harian](#11-rutinitas-harian-setiap-buka-termux)
12. [Folder kerja dan lokasi file](#12-folder-kerja-dan-lokasi-file)
13. [Keamanan dan hemat kuota](#13-keamanan-dan-hemat-kuota)
14. [Troubleshooting](#14-troubleshooting)
15. [Referensi](#15-referensi)

---

## 1. Baca dulu: kenapa lewat proot Ubuntu?

Dokumentasi resmi Hermes punya jalur khusus Termux lewat repositori APT. Namun, saat panduan ini ditulis, halaman resminya menandai paket Termux **sedang bermasalah** dan menyarankan menunggu perbaikan. Cek halaman [Android / Termux](https://hermes-agent.nousresearch.com/docs/getting-started/termux) lebih dulu. Kalau sudah diperbaiki, jalur resmi itu lebih mudah.

Panduan ini memakai jalur alternatif yang berhasil diuji: menjalankan **Ubuntu di dalam Termux** (proot-distro), lalu memasang Hermes dari kode sumbernya.

**Yang perlu disiapkan:**

- HP Android dengan prosesor **arm64**. Cek nanti dengan `uname -m` (harus `aarch64`).
- Ruang penyimpanan kosong yang cukup (Ubuntu, Python, dan dependensi cukup besar).
- Internet yang stabil. Beberapa langkah mengunduh dan membangun paket cukup lama.
- Akun Google untuk membuat API key gratis (Gemini), atau akun Nous Portal.

---

## 2. Alur singkat

```text
Termux (F-Droid)
   └── proot-distro  →  Ubuntu
          └── Python 3.14 + venv
                 └── Hermes Agent
                        └── Provider model (API key)  →  hermes --tui
```

---

## 3. Langkah 1: Pasang Termux

1. Unduh **Termux dari F-Droid** (f-droid.org). Jangan pakai versi Play Store karena tidak lagi diperbarui.
2. Buka Termux, lalu perbarui paket:

```bash
pkg update && pkg upgrade -y
```

Kalau ada pertanyaan `Y/n` atau soal konfigurasi, tekan **Enter** untuk memilih default.

3. Cek arsitektur HP:

```bash
uname -m
```

Hasil harus `aarch64`.

---

## 4. Langkah 2: Pasang Ubuntu (proot-distro)

Jalankan di Termux:

```bash
pkg install proot-distro -y
proot-distro install ubuntu
proot-distro login ubuntu
```

Kalau berhasil, tampilan prompt berubah menjadi seperti `root@localhost:~#`. Itu tandanya Anda sudah **di dalam Ubuntu**. Semua langkah berikutnya dijalankan di dalam Ubuntu, kecuali dikatakan lain.

---

## 5. Langkah 3: Siapkan Ubuntu

Pasang alat dasar dan pustaka yang dibutuhkan untuk membangun paket Python (misalnya `pillow`):

```bash
apt update && apt upgrade -y
apt install -y git curl nano build-essential software-properties-common \
  libjpeg-dev libpng-dev zlib1g-dev libfreetype6-dev
```

---

## 6. Langkah 4: Pasang Python 3.14

Hermes versi ini membutuhkan **Python 3.14**. Python bawaan Ubuntu (misalnya 3.12) bisa menyebabkan error ketidakcocokan versi. Gunakan PPA *deadsnakes*:

```bash
add-apt-repository -y ppa:deadsnakes/ppa
apt update
apt install -y python3.14 python3.14-venv python3.14-dev
```

Cek hasilnya:

```bash
python3.14 --version
```

Harus muncul `Python 3.14.x`.

---

## 7. Langkah 5: Ambil kode Hermes

```bash
mkdir -p ~/.hermes
cd ~/.hermes
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
```

Panduan ini memakai lokasi `~/.hermes/hermes-agent`. Semua perintah selanjutnya mengasumsikan lokasi ini.

---

## 8. Langkah 6: Buat venv dan pasang Hermes

**Apa itu venv?** Folder Python terisolasi khusus untuk proyek ini. Ubuntu terbaru melarang `pip install` langsung ke sistem (error `externally-managed-environment`), jadi venv adalah cara yang benar.

```bash
python3.14 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -e '.[termux]'
```

Penjelasan:

- `source venv/bin/activate` mengaktifkan venv. Prompt akan diawali `(venv)`.
- `.[termux]` adalah paket tambahan yang dipilih khusus untuk Android. Menurut README resmi, ekstra lengkap `.[all]` menarik dependensi suara yang tidak kompatibel dengan Android.
- Selama venv aktif, Anda **tidak perlu** flag `--break-system-packages`.

Proses ini bisa lama. Tunggu sampai kembali ke prompt tanpa error, lalu cek:

```bash
hermes --version
hermes doctor
```

`hermes doctor` memeriksa masalah umum.

---

## 9. Langkah 7: Pilih provider model

Hermes butuh "otak", yaitu model AI dari sebuah provider. Jalankan wizard:

```bash
hermes model
```

Pilih salah satu opsi berikut.

### Opsi A: Google Gemini lewat API key (bisa gratis)

1. Buka **aistudio.google.com**, login dengan akun Google, lalu buat **API key**.
2. **Cek tier key Anda** di halaman API keys. Pastikan tertulis **Free tier**. Kalau project-nya memakai billing, pemakaian bisa ditagih.
3. Di wizard `hermes model`, pilih Google Gemini (AI Studio), lalu tempel key saat diminta `GOOGLE_API_KEY`. Di Termux, tempel dengan menekan lama layar. Tampilan akan berupa bintang (`****`), itu normal.
4. Saat diminta **Base URL**, tekan **Enter** (default sudah benar).
5. Pilih model, misalnya `gemini-flash-lite-latest`. Hindari model `pro` di free tier karena kuotanya lebih ketat.
6. Pilih **reasoning effort** `low` untuk menghemat kuota.

> Wizard mungkin menampilkan `Tier check: paid` walau key Anda free. Sumber yang benar adalah halaman API keys di AI Studio, jadi cocokkan di sana.

### Opsi B: Nous Portal

```bash
hermes setup --portal
```

Perintah ini login lewat OAuth (browser) dan menjadikan Nous Portal sebagai provider. Ada paket gratis dengan pilihan model terbatas, tetapi ketentuannya bisa berubah. Cek sendiri di portal.nousresearch.com. Untuk memeriksa hasilnya:

```bash
hermes portal info
```

Jika browser tidak terbuka otomatis di proot, salin URL dari terminal dan buka manual di browser HP. Kalau tetap macet, pakai Opsi A.

### Di mana pengaturan tersimpan?

| File | Isi |
| --- | --- |
| `/root/.hermes/config.yaml` | Pengaturan umum |
| `/root/.hermes/.env` | API key (**rahasia**) |

Ubah satu nilai dengan `hermes config set <key> <value>`, atau edit manual dengan `nano`.

---

## 10. Langkah 8: Jalankan Hermes

Buat folder kerja terpisah dulu (lihat [bagian 12](#12-folder-kerja-dan-lokasi-file)), lalu jalankan:

```bash
mkdir -p ~/workspace
cd ~/workspace
hermes --tui
```

**TUI** (*Text User Interface*) adalah tampilan interaktif di dalam terminal. Mesin agennya terpisah dari tampilan, jadi pengaturan Anda tidak hilang saat keluar.

Tunggu status `summoning hermes…` berganti `ready`, lalu coba:

```text
halo, sebutkan model apa yang kamu pakai dan jawab dalam 2 kalimat
```

**Tes agent (bukan sekadar chatbot):**

```text
Buat file hitung.py berisi fungsi rata-rata dari list angka, jalankan dengan [4, 8, 15], lalu tampilkan hasilnya.
```

Kalau agen membuat file, menjalankannya, dan menampilkan `9.0`, berarti agen berfungsi penuh.

**Perintah di dalam TUI:**

| Perintah | Fungsi |
| --- | --- |
| `/help` | Daftar perintah |
| `/model` | Ganti model |
| `/new` | Mulai percakapan baru |
| `/usage` | Cek pemakaian |
| `Ctrl+C` | Menghentikan pekerjaan agen yang sedang berjalan |

Untuk **keluar**, coba `/exit` atau `Ctrl+D`. Kombinasi ini belum diverifikasi pada semua versi, jadi cek `/help` kalau tidak berhasil.

---

## 11. Rutinitas harian (setiap buka Termux)

Setiap Termux ditutup, sesi Ubuntu ikut berhenti. Untuk memulai lagi:

```bash
proot-distro login ubuntu
cd ~/.hermes/hermes-agent
source venv/bin/activate
cd ~/workspace
hermes --tui
```

**Opsional: pintasan.** Di dalam Ubuntu, jalankan sekali:

```bash
echo "alias hh='cd ~/.hermes/hermes-agent && source venv/bin/activate && cd ~/workspace && hermes --tui'" >> ~/.bashrc
source ~/.bashrc
```

Setelah itu, cukup `proot-distro login ubuntu`, lalu ketik `hh`.

---

## 12. Folder kerja dan lokasi file

- Agen biasanya menulis file ke **folder tempat Anda menjalankannya**. Karena itu jalankan dari `~/workspace`, **bukan** dari folder `hermes-agent`, agar kode program Hermes tidak ikut berubah.
- Cek isinya: `ls ~/workspace`.
- Cari file yang hilang: `find / -name "nama_file.py" 2>/dev/null`.

**Lokasi fisik di Android.** Folder `root` Ubuntu sebenarnya folder biasa milik Termux:

```text
/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/root/workspace
```

File di sana **tidak muncul** di galeri atau File Manager. Untuk menyalinnya ke penyimpanan HP, jalankan di **Termux** (bukan di Ubuntu; keluar dulu dengan `exit`):

```bash
termux-setup-storage
cp $PREFIX/var/lib/proot-distro/installed-rootfs/ubuntu/root/workspace/hitung.py ~/storage/downloads/
```

---

## 13. Keamanan dan hemat kuota

- **API key itu rahasia.** Jangan menempelkannya di chat, screenshot, atau repositori GitHub. Jangan pernah meng-upload file `.env`.
- **Agen bisa menjalankan kode dan menulis file** (tool seperti `execute_code` dan `write_file`). Mulai dengan tugas kecil di `~/workspace`, dan baca permintaan persetujuan perintah sebelum menyetujuinya.
- **Free tier punya batas.** Satu tugas agen bisa memicu banyak panggilan model, jadi error `429` (kuota habis) wajar. Tunggu beberapa menit atau pakai model yang lebih ringan.
- **Jangan kirim data sensitif** lewat agen di free tier. Syarat Google dapat mengizinkan input dipakai untuk pengembangan produk, jadi cek ketentuan terbarunya di AI Studio.
- **Hati-hati dengan `hermes update`.** Lingkungan di proot rapuh (versi Python dan dependensi). Sebelum update, salin config: `cp ~/.hermes/config.yaml ~/.hermes/config.yaml.bak`. Setelah update, jalankan `hermes doctor`.

---

## 14. Troubleshooting

| Gejala | Penyebab | Solusi |
| --- | --- | --- |
| Gagal build `pillow` | Pustaka gambar belum terpasang | `apt install -y libjpeg-dev libpng-dev zlib1g-dev libfreetype6-dev`, lalu ulangi `pip install` |
| `externally-managed-environment` | `pip` dijalankan di luar venv | `source venv/bin/activate`, lalu ulangi |
| `No module named 'ruamel'` | Modul belum terpasang | `pip install ruamel.yaml` (di dalam venv) |
| Error ketidakcocokan versi Python (misal 3.12 vs 3.14) | venv dibuat dengan Python yang salah | `rm -rf venv && python3.14 -m venv venv && source venv/bin/activate && pip install -e '.[termux]'` |
| `hermes: command not found` | venv belum aktif | `cd ~/.hermes/hermes-agent && source venv/bin/activate` |
| `429` / kuota habis | Batas free tier | Tunggu, kurangi request, pakai model flash/lite |
| `Tier check: paid` | Deteksi tier tidak cocok | Cek tier key di AI Studio (halaman API keys) |
| Macet di `summoning hermes…` | Agen masih memuat atau ada error | Tunggu sebentar, lalu `hermes doctor` |
| Login browser Nous Portal macet | OAuth di proot | Buka URL manual di browser HP, atau pakai API key (Opsi A) |

Kalau error tidak ada di tabel: jalankan `hermes doctor`, lalu salin pesan error **lengkap** (tanpa API key) untuk dicari di [GitHub Issues](https://github.com/NousResearch/hermes-agent/issues) atau ditanyakan ke komunitas.

---

## 15. Referensi

- Repositori resmi: <https://github.com/NousResearch/hermes-agent>
- Dokumentasi: <https://hermes-agent.nousresearch.com/docs/>
- Panduan Android / Termux resmi: <https://hermes-agent.nousresearch.com/docs/getting-started/termux>
- Referensi perintah CLI: <https://hermes-agent.nousresearch.com/docs/reference/cli-commands>
- Nous Portal: <https://portal.nousresearch.com>
- Google AI Studio: <https://aistudio.google.com>

---

*Dibuat dari pengalaman pemasangan langsung di Android. Kalau langkah di sini sudah tidak cocok dengan versi terbaru, ikuti dokumentasi resmi.*
