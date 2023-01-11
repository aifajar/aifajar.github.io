---
title: Metode Serangan Siber terhadap Password
date: 2023-01-11 23:59:59 +0700
categories: [Cybersecurity, CompTIA Security+]
tags: [cybersecurity,comptia security+,password attack]
---

*Password* dan *username* merupakan komponen akun yang digunakan untuk proses autentikasi. Autentikasi merupakan langkah awal yang dibutuhkan pengguna agar dapat mengakses sistem yang sesuai dengan wewenangnya. *Password* yang dipilih oleh pengguna akan dikonversi dan disimpan ke bentuk nilai *hash*. Secara teori, *password* tidak bisa diketahui selain oleh penggunanya, karena konversi ke nilai *hash* bersifat satu arah (*one-way*) dan tidak dapat dikembalikan ke versi *plaintext*-nya (orisinil, sebelum *password* diubah menjadi nilai *hash*).

Bagi *hacker*, serangan terhadap *password* bertujuan agar dapat masuk dan mengakses suatu sistem secara tidak sah (*unauthorized access*) melalui akun pengguna. Berikut merupakan beberapa metode serangan yang digunakan *hacker* untuk mendapatkan *password* dari suatu akun.

## Spraying Attack
*Spraying attack* merupakan metode serangan yang menggunakan berbagai *password* umum (tidak aman) sebagai tebakan dan mecobanya ke beberapa *username*. Contoh *password* yang digunakan sebagai tebakan adalah <mark style="background-color: #808080">qwerty</mark> atau <mark style="background-color: #808080">password</mark>.

## Dictionary Attack
*Dictionary attack* merupakan metode serangan menggunakan setiap kata-kata umum yang terdapat pada kamus sebagai *password* tebakan, lalu mencocokkan nilai *hash*-nya dengan yang tersimpan di sistem. Metode ini cocok untuk meretas *password* yang tidak kompleks (tidak memiliki karakter angka atau spesial).

## Brute Force Attack 
*Brute force attack* merupakan metode serangan yang menggunakan setiap kemungkinan kombinasi dari karakter yang ada sebagai *password* tebakan, lalu mencocokkan nilai *hash*-nya dengan yang tersimpan di sistem. *Brute force* cocok untuk meretas *password* yang jumlah karakternya pendek. Metode serangan ini bisa membutuhkan waktu yang sangat lama jika *password* yang ingin diretas memiliki jumlah karakter yang panjang. Untuk mempercepat waktu yang dibutuhkan, *hacker* dapat mengakalinya dengan meningkatkan komputasi sumber daya yang digunakan.

### Offline Attack
*Offline attack* merupakan metode serangan dimana *hacker* berhasil mengambil *file* atau *database* yang berisi kumpulan nilai *hash* dari *password* yang ada di sistem. Setelah mendapatkan *file* tersebut, *hacker* mencoba mencari versi *plaintext* dari nilai *hash* tersebut secara *offline*. Contoh *file* yang dimaksud adalah <mark style="background-color: #808080">/etc/shadow</mark> pada sistem operasi Linux. Metode ini tidak berinteraksi dengan sistem autentikasi.

### Online Attack
*Online attack* merupakan metode serangan dimana *hacker* berinteraksi secara langsung dengan sistem autentikasi. Contoh sistem autentikasi yang ada antara lain *form login* situs web dan VPN *gateway*. *Password* bisa didapatkan secara *online* melalui forum yang menjual kebocoran data ataupun melalui peretasan secara *offline*.

## Rainbow Table Attack
*Rainbow table attack* merupakan metode serangan yang menggunakan *precomputed table* yang berisi pasangan semua kemungkinan *password* dan nilai *hash*-nya. *Precomputed* yang dimaksud adalah nilai *hash* sudah dikalkulasi sebelumnya, sehingga telah tersedia sebelum serangan dilancarkan. Ketika nilai *hash* dari *password* yang ingin diretas berada dalam *precomputed table*, maka versi *plaintext* akan muncul di hasil pencarian.

## Plaintext / Unencrypted Attack
*Plaintext / Unencrypted Attack* merupakan metode serangan yang mengeksploitasi kerentanan pada penyimpanan (*storage*) atau protokol autentikasi jaringan yang tidak menggunakan enkripsi saat proses transmisi. Beberapa contoh protokol autentikasi jaringan yang dimaksud adalah PAP, Telnet dan autentikasi dasar pada HTTP/FTP. *Password* juga tidak boleh disimpan pada sebuah *file* yang tidak terenkripsi. Salah satu contoh populer dari dari metode serangan ini adalah adanya informasi *password* yang terdapat pada *source code* yang disimpan di repositori publik seperti GitHub.

## Hybrid Attack
*Hybrid attack* menggunakan kombinasi antara serangan *brute force* dan serangan *dictionary*. Metode ini mengkombinasikan kata umum yang ada di kamus atau nama seseorang dengan karakter lain selain huruf seperti angka dan spesial. Algoritma yang diterapkan, ditentukan oleh *hacker* yang sudah memiliki beberapa informasi terkait pengguna yang menjadi targetnya. Contoh *password* yang bisa dipecahkan dengan metode ini adalah <mark style="background-color: #808080">pa55w0rd</mark> atau <mark style="background-color: #808080">j4mes!</mark>.