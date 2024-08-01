---
title: Special Permissions di Linux
date: 2024-08-01 23:59:59 +0700
categories: [Linux, RHCSA]
tags: [suid, sgid, sticky bits]
---

Pada Linux terdapat beberapa special permissions yang biasanya tidak kita *set* secara *default*. Penggunaan *permissions* tipe ini digunakan untuk skenario tertentu. Terdapat 3 jenis *special permissions* pada Linux, yaitu SUID, SGID, dan sticky bit.

## SUID
SUID merupakan singkatan dari set user ID. SUID memungkinkan *user* untuk membaca, memodifikasi dan/atau mengeksekusi *file/s* yang memang tidak memiliki akses terhadapnya. *User* seolah-olah dianggap sebagai *owner* dari *file* tersebut. Contoh penerapannya adalah *command* **passwd** yang secara *default* bisa diakses oleh semua *users* untuk mengganti *password* mereka.

![img-description](/assets/img/posts/special-permissions-di-linux/suid.png)
_SUID pada File Passwd_

Penerapan SUID memiliki risiko besar terkait *security* karena bisa memberikan akses root ke suatu *file*. Ada baiknya SUID tidak pernah diterapkan kepada *file/s* selain kepada *file/s* terkait sistem operasi yang memang sudah terpasang *default*.

## SGID
SGID merupakan singkatan dari set group ID. SGID memungkinkan *users* untuk membaca, memodifikasi dan/atau mengeksekusi *file/s* yang memang tidak memiliki akses terhadapnya. *User* seolah-olah dianggap sebagai *group owner* dari *file/s* atau direktori tersebut.

Ketika *user* mengakses *file/s* atau direktori yang sudah terpasang SGID, maka akses tersebut dapat dijalankan karena adanya akses dari *group owner file* tesebut bukan dari akses *group user* tersebut. Hal ini sangat berguna jika kita tidak perlu mengubah *group* dari setiap *user* untuk mengakses *file/s* atau direktori tertentu. Jika SGID diterapkan pada level direktori, maka semua *file* yang dibuat akan mewarisi *group owner* dari direktori tersebut.

## Sticky Bit
Sticky bit sangat berguna bila kita ingin melindungi *file/s* dari kejadian tidak disengaja dimana *file/s* tersebut berada di direktori yang memiliki konfigurasi banyak *users* memiliki akses w (write). Ketika sticky bit diterapkan, *user* bisa menghapus *file/s* hanya ketika memenuhi kondisi berikut.
- Memiliki akses root.
- Merupakan *owner* dari *file/s* tersebut.
- Merupakan *owner* dari direktori *file/s* tersebut.

## Penerapan Special Permissions
Penerapan *special permissions* pada Linux dapat diterapkan dengan 2 mode pada *command* **chmod**. Mode pertama menggunakan mode numerik dan yang kedua menggunakan mode relatif.

### Mode Numerik
Jika kita ingin menggunakan mode numerik untuk penerapan *special permissions*, maka kita perlu menambahkan 4 digit argumen pada *command* chmod. Digit pertama merujuk ke *special permissions*, sedangkan 3 digit berturut-turut setelahnya merupakan *basic permissions* berupa akses rwx untuk *owner*, *group*, dan *other users*.

Contoh:
```sh
chmod 4644 example.txt
```
*File* example.txt memiliki *special permissions* SUID dan rw untuk *owner*, r untuk *group*, dan r untuk *other user/s*.

### Mode Relatif

Menggunakan chmod mode numerik memiliki risiko yang besar karena ada kemunngkinan terjadinya *overwriting* pada *permissions* sebelumnya. *Special permissions* dapat diterapkan menggunakan mode relatif dimana konfigurasi *special permissions* ditambahkan ke *permissions* yang sudah ada di dalam *file* atau direktori.
- SUID menggunakan chmod u+s.
- SGID menggunakan chmod g+s.
- Sticky bit menggunakan chmod +t.

Contoh: 
Direktori bernama example memiliki *permissions* rwx untuk *owner*, rx untuk *group*, dan rx untuk *other user/s*. Kita perlu menerapkan SGID di direktori tersebut, maka kita bisa menjalankan *command* berikut.
```sh
chmod g+s example
```

### Verifikasi
Untuk melakukan verifikasi pada *special permissions* yang sudah diterapkan, kita dapat menggunakan *command* ls -ld. Berikut tampilan dari *special permissions* yang sudah diterapkan.
- SUID ditunjukkan oleh huruf s atau S pada posisi *permission* x di *user owner*.
![img-description](/assets/img/posts/special-permissions-di-linux/ss_1.png)
_Verifikasi Penerapan SUID_

- SGID ditunjukkan oleh huruf s atau S pada posisi *permission* x di *group*.
![img-description](/assets/img/posts/special-permissions-di-linux/ss_2.png)
_Verifikasi Penerapan SGID_

- Sticky bit ditunjukkan oleh huruf t atau T pada posisi *permission* x di *others*.
![img-description](/assets/img/posts/special-permissions-di-linux/ss_3.png)
_Verifikasi Penerapan Sticky Bit_

Huruf s atau S maupun t atau T bergantung dari *permission* awal e (*execute*) dari suatu *file* atau direktori. Jika terdapat *permission* e maka ditunjukkan dengan huruf kecil, jika tidak ada *permission* e maka ditunjukkan dengan huruf besar.

## Perbandingan Special Permissions

| Permissions | Mode Numerik | Mode Relatif | Efek ke File | Efek ke Direktori |
|---|---|---|---|---|
| SUID | 4 | u+s | *User/s* mengeksekusi *file* dengan akses *permissions* dari *file owner*. | Tidak berefek. |
| SGID | 2 | g+s | *User/s* mengeksekusi *file* dengan akses *permissions* dari *group owner*. | *File/s* yang dibuat di direktori terkait memiliki *group owner* yang sama. |
| Sticky bit | 1 | +t | Tidak berefek. | Mencegah *user/s* untuk menghapus *file/s* yang dimiliki oleh *user* atau *group* lain. |


