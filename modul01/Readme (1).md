# Laporan Praktikum Sistem Operasi
## Modul 1–3: Persiapan Lingkungan, Arsitektur, dan Menjelajahi Xinu OS

### Data Praktikan
| Keterangan | Isi |
|------------|-----|
| **Nama** | Moreno Maheswara Anggoro |
| **NIM** | 108072500044 |
| **Kelas** | IF-05-04 |
| **Asisten Praktikum** | [Isi nama asisten] |
| **Tanggal Praktikum** | [Isi tanggal praktikum] |

---

## 1. Tujuan
1. Mengenal tata tertib laboratorium serta menyiapkan perangkat yang dipakai selama praktikum, yaitu VirtualBox, Ubuntu, Xinu OS, dan Sourcetrail.
2. Memahami konsep *cross-development* pada Xinu, yaitu pembagian peran antara *Development-System VM* dan *Backend VM*.
3. Dapat mengompilasi kode sumber Xinu, lalu menjalankan hasilnya di mesin target lewat jaringan menggunakan PXE dan TFTP.
4. Dapat terhubung ke konsol Xinu melalui *serial port* dengan Minicom dan mencoba beberapa perintah dasar pada *shell*-nya.

---

## 2. Landasan Teori
Xinu adalah sistem operasi berukuran kecil yang dirancang untuk perangkat *embedded*. Cara pengembangannya mengikuti pola **cross-development**: kode ditulis dan dikompilasi di komputer biasa (*host*), kemudian *image* yang dihasilkan dikirim ke mesin target (*backend*) untuk dijalankan di sana.

Praktikum ini memakai dua *Virtual Machine* dengan peran berbeda:

1. **Development-System VM**: menjalankan Debian Linux dan menyimpan kode sumber Xinu, *compiler*, serta layanan DHCP dan TFTP.
2. **Backend VM**: mesin target tanpa OS yang melakukan *boot* lewat jaringan (PXE). Ia mengambil *image* Xinu dari server TFTP lalu menjalankannya.

Kedua VM disambungkan oleh *Virtual Serial Port*. Lewat jalur ini, perintah dapat diketik dari terminal Development-System dan diteruskan ke Xinu di Backend.

---

## 3. Langkah Kerja dan Hasil

### 3.1 Login dan Kompilasi Xinu
Seluruh langkah awal dikerjakan di **Development-System VM**. Setelah VM menyala, *login* ke Debian dengan akun berikut:
- **Username:** `xinu`
- **Password:** `xinurocks`

Di terminal, masuk ke folder `compile` milik Xinu, bersihkan hasil *build* lama, lalu kompilasi ulang agar *image* yang dipakai benar-benar versi terbaru.
```bash
$ cd xinu/compile
$ make clean
$ make
```
**Hasil:**
Perintah `make` mengubah kode sumber C menjadi *image* `xinu.elf`. Setelah selesai, *image* tersebut otomatis disalin ke folder server TFTP (`/srv/tftp/xinu.boot`), sehingga bisa langsung diambil oleh Backend VM.

![Terminal Compile Xinu](assets/dummy.png)

*Gambar 1: Hasil kompilasi Xinu dengan `make` di Development-System VM.*

### 3.2 Booting Backend VM lewat PXE
Berikutnya, **Backend VM** dinyalakan. Mesin ini tidak punya disk berisi sistem operasi, jadi ia melakukan *boot* dari jaringan. Urutan yang terlihat:
1. Layar **GRUB** muncul lebih dulu.
2. Backend VM meminta alamat IP ke server DHCP di Development-System VM.
3. File `xinu.boot` diunduh melalui TFTP.
4. Xinu dimuat ke memori dan mulai berjalan.

![Booting Backend VM](assets/dummy.png)

*Gambar 2: Backend VM saat *boot* lewat jaringan (PXE) dan menampilkan GRUB.*

### 3.3 Menyambung ke Xinu dengan Minicom
Supaya bisa berinteraksi dengan Xinu di Backend VM, kembali ke terminal **Development-System VM** dan jalankan `minicom`, aplikasi komunikasi serial. Karena aplikasi ini mengakses perangkat secara langsung, perintahnya perlu dijalankan dengan `sudo`.
```bash
$ sudo minicom
```
*(Password: `xinurocks`)*

**Hasil:**
Terminal kini tersambung ke konsol Xinu. Prompt yang semula `xinu@xinu-develop-end:$` berubah menjadi **`xsh$`**, tanda bahwa sesi sudah berada di dalam *Xinu Shell*.

![Koneksi Minicom](assets/dummy.png)

*Gambar 3: Minicom berhasil tersambung dan menampilkan prompt `xsh$`.*

### 3.4 Mencoba Perintah pada Xinu Shell
Dari prompt `xsh$`, perintah diterima langsung oleh kernel Xinu, dan daftarnya tidak sama dengan perintah Linux. Untuk mengetahui apa saja yang tersedia, dijalankan perintah `help`.

```bash
xsh$ help
```

![Perintah Help Xinu](assets/dummy.png)

*Gambar 4: Daftar perintah bawaan Xinu yang ditampilkan oleh `help`.*

Selain itu, perintah `ls` dan `cd` dicoba untuk melihat struktur sistem berkas Xinu yang sederhana, dan perintah untuk mematikan sistem seperti `halt` atau `shutdown` juga dicoba.

---

## 4. Pembahasan
Dari rangkaian percobaan di atas, ada beberapa hal yang bisa dicatat:

1. **Pemisahan host dan target.**
   Penggunaan dua VM menggambarkan cara kerja pengembangan sistem *embedded* di dunia nyata, misalnya pada perangkat IoT atau *microcontroller*. Perangkat target biasanya punya sumber daya terbatas, sehingga kompilasi dilakukan di komputer *host*, dan hanya hasil *binary*-nya yang dikirim ke target.
2. **Cara kerja network booting.**
   Backend VM tidak memerlukan penyimpanan lokal. Saat menyala, kartu jaringan virtualnya mencari server DHCP. Development VM menjawab dengan memberi alamat IP sekaligus menunjukkan lokasi server TFTP. Setelah itu `xinu.boot` diunduh dan langsung dijalankan dari RAM.
3. **Fungsi serial port dan Minicom.**
   Backend VM berjalan tanpa tampilan grafis, sehingga *input* dan *output* lewat *serial port*. Minicom berperan sebagai *terminal emulator* yang meneruskan ketikan dari Development VM ke konsol Xinu di Backend VM.
4. **Xinu Shell (`xsh$`).**
   Shell Xinu (`shell.c`) dibuat sangat ringkas: ia membaca teks yang diketik, mengambil nama perintahnya, lalu memanggil fungsi kernel yang sesuai. Perintah seperti `help`, `uptime`, atau `kill` dijalankan langsung di kernel, karena Xinu tidak memisahkan *user space* dan *kernel space* seketat Linux modern.

---

## 5. Kesimpulan
1. Tata tertib laboratorium sudah dipahami dan lingkungan *cross-development* berhasil disiapkan dengan Oracle VM VirtualBox.
2. Xinu membutuhkan dua mesin: **Development-System** untuk kompilasi serta layanan DHCP dan TFTP, dan **Backend** sebagai tempat *image* Xinu dijalankan.
3. *Image* hasil `make` berhasil di-*boot* oleh Backend VM melalui jaringan (PXE).
4. *Xinu Shell* (`xsh$`) berhasil diakses lewat Minicom dan beberapa perintah dasarnya sudah dicoba. Pemahaman ini menjadi bekal untuk modul berikutnya, yaitu membaca kode sumber, manajemen proses, dan *system call*.
