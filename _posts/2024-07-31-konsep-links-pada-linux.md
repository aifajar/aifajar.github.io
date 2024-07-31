---
title: Konsep Links pada Linux
date: 2024-07-31 23:59:59 +0700
categories: [Linux, RHCSA]
tags: [links, hard links, soft links]
---

*Links* pada Linux merupakan cara untuk membuat referensi atau alias ke suatu *file* atau direktori (folder) lain. Terdapat dua jenis *links* pada Linux, yaitu *hard links* maupun *symbolic* (*soft*) *links*. Untuk memahami *links* pada Linux, kita perlu mengetahui tentang inode. **Inode** merupakan sebuah struktur data yang digunakan pada Linux untuk meyimpan semua informasi metadata pada *file* kecuali nama *file*. Berikut beberapa informasi penting pada *file* yang tersimpan pada inode.
- *Timestamps*: Informasi terkait waktu kapan *file* dibuat serta kapan *file* terkahir kali diakses dan dimodifikasi.
- *Pointers to data block*: Lokasi blok *disk* dimana *file* sebenarnya disimpan.
- Tipe *file*: Beberapa tipe diantaranya adalah *file* reguler, direktori, *symbolic links*, dsb.
- *File Owner*: Informasi *user* ID (UID) dan *group* ID (UID) dari pemilik *file*.
- *Permission*: Hak akses *read*, *write*, dan *execute* pada *file* yang ditujukan ke *owner*, *group*, dan *other*.

## Cara Kerja Inode

Saat sebuah *file* dibuat, sistem *file* pada Linux akan mengalokasikan sebuah inode dan kumpulan blok data pada *file* tersebut. Inode berisi metadata sedangkan kumpulan blok data digunakan untuk menyimpan konten data dari *file*. Ketika kita mengakses *file*, sistem *file* pada Linux akan menggunakan direktori untuk memetakan antara nama *file* dengan inode dari *file* tersebut. Informasi nama *file* tersimpan di dalam direktori. Inode tidak menyimpan informasi metadata nama *file* tetapi tau berapa banyak nama *file* yang terhubung ke satu inode. Kumpulan dari nama-nama *file* yang terhubung ke satu inode inilah yang dinamakan **hard links**.

## Hard Links

Ketika kita membuat sebuah *file* dan memberikan nama pada *file* tersebut, bisa dikatakan bahwa itu juga merupakan sebuah *hard link*. Pada sistem Linux, beberapa *hard links* bisa dibuat ke dalam sebuah file. *File* asli dan *file* target pada *hard links* memiliki inode yang sama. Hal ini biasanya diperlukan ketika kita memerlukan sebuah file dengan konten yang sama di beberapa lokasi atau *path*. Ketika terjadi perubahan di salah satu *file hard links*, maka perubahan juga akan terjadi di *file/s* lainnya. Kondisi ini bisa terjadi karena semua *hard links* mengarah ke blok data yang sama pada *disk*.

Ada beberapa batasan pada penerapan *hard links*, diantaranya:
- Semua *hard links* harus berada dalam satu *device* (*partition*, *logical volume*, dsb) yang sama.
- Tidak bisa diterapkan ke direktori.
- Ketika *file* terakhir pada *hard link* terhapus maka akses ke konten data pada *file* juga terhapus.

Jika salah satu *file* pada *hard links* terhapus maka data pada *file/s* lainnya tidak berdampak. *Hard links* dapat diterapkan dengan menjalankan *command* ln.

Contoh:
```
ln original.txt hardlink.txt
```
- hardlink.txt merupakan sebuah *hard link* dari *file* original.txt.

## Symbolic Links

**Symbolic link** atau yang sering disebut juga sebagai *soft link* tidak terhubung langsung ke inode, namun terhubung ke nama *file*. Keuntungan menggunakan *symbolic link* adalah terkait fleksibilitas, dapat menhubungkan *link files* beda *devices* dan juga direktori. Kelemahan terbesar dari penggunaan *symbolic link* adalah ketika *file* asli dipindahkan lokasinya atau dihapus maka *link* menjadi tidak valid dan eror. *Symbolic link* dapat diterapkan dengan menjalankan *command* ln -s.

Contoh: 
```
ln original.txt softlink.txt
```
- softlink.txt merupakan sebuah *symbolic link* dari *file* original.txt.

Berikut merupakan gambaran ilustrasi hubungan antara inode, *hard links*, maupun *soft links*.
![img-description](/assets/img/posts/konsep-links-pada-linux/ilustrasi.png)
_Ilustrasi Hubungan antara Inode, Hard Links, dan Soft Links_

## Demo Links pada Linux

1. Membuat direktori dan *file*.
 ```
mkdir demo
cd demo
echo "Original:v1" > original.txt
cat original.txt
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_1.png)
_Screenshot Langkah 1_

2. Membuat *hard link file* hardlink.txt yang merujuk ke *file* original.txt.
 ```
ln original.txt hardlink.txt
```
3. Verifikasi *hard link*. Pastikan konten data dari *file* hardlink.txt dan original.txt sama, begitu juga dengan nomor inode-nya.
 ```
cat hardlink.txt
ls -li
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_2.png)
_Screenshot Langkah 3_

4. Ganti konten dari *file* hardlink.txt. Pastikan konten dari *file* original.txt juga ikut berubah.
 ```
echo "Original:v2" > hardlink.txt
cat hardlink.txt
cat original.txt
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_3.png)
_Screenshot Langkah 4_

5. Membuat *symbolic link file* softlink.txt yang merujuk ke *file* original.txt.
```
ln -s original.txt softlink.txt
```
6. Verifikasi *soft link*. Pastikan konten data dari *file* soflink.txt dan original.txt sama, namun nomor inode kedua *files* berbeda.
```
cat softlink.txt
ls -li
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_4.png)
_Screenshot Langkah 6_

7. Ganti konten dari *file* softlink.txt. Pastikan konten dari *file* original.txt juga ikut berubah.
```
echo "Original:v3" > softlink.txt
cat softlink.txt
cat original.txt
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_5.png)
_Screenshot Langkah 7_

8. Hapus *file* original.txt.
```
rm original.txt
```
9. Verifikasi konten dari *file* hardlink.txt dan softlink.txt. Pastikan *file* hardlink.txt masih memiliki konten data sedangkan *file* softlink.txt mengalami eror.
```
cat hardlink.txt
cat softlink.txt
ls -li
```
![img-description](/assets/img/posts/konsep-links-pada-linux/ss_6.png)
_Screenshot Langkah 9_