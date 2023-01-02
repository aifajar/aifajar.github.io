---
title: Mengenal Teknologi RAID
date: 2023-01-02 23:59:59 +0700
categories: [Storage, CompTIA Security+]
tags: [storage,comptia security+,raid]
---

RAID (*redundant array of independent disks*) merupakan teknologi yang digunakan untuk menggabungkan dan mengatur beberapa perangkat *storage* sebagai satu kesatuan *logic* untuk mencapai tujuan tertentu. Sistem operasi akan menganggap sebuah RAID yang terdiri dari kumpulan (*array*) *physical drives* sebagai sebuah *logical drive*. Penggunaan RAID menawarkan tujuan penyimpanan data melalui beberapa perangkat *storage* untuk meningkatkan *availability*, *reliability*, *capacity*, *redundancy*, dan/atau *performance*.

## Manfaat RAID

Berikut beberapa manfaat yang dapat diberikan ketika menggunakan RAID:

### Availability
Sistem bisa diakses dan selalu tersedia bagi pengguna yang membutuhkan.
### Reliability
*Reliability* merupakan kemampuan suatu perangkat untuk bekerja sebagaimana mestinya dalam periode waktu yang dapat diprediksi.
### Capacity
*Capacity* atau kapasitas merupakan total ukuran data yang bisa disimpan pada perangkat *drive*. Kapasitas total pada RAID juga bergantung terhadap jumlah *physical drive* (N) yang tersedia dalam penyimpanan data.
### Redundancy
*Redundancy* dapat meningkatkan *fault tolerance* sebuah sistem. *Fault tolerance* adalah kemampuan sistem untuk tetap beroperasi ketika satu atau banyak komponen sistem mengalami kegagalan atau kerusakan. Salah satu bentuk penerapan *redundancy* adalah dengan menyiapkan perangkat *storage backup* tersendiri yang berbeda dengan perangkat *storage* yang digunakan untuk lingkungan produksi. Perangkat *backup* tersebut dapat menggantikan perangkat di lingkungan produksi ketika mengalami kegagalan secara cepat.
### Performance
*Performance* atau performa adalah ukuran ketika suatu operasi dijalankan. Bagi perangkat *storage*, ukuran performa yang umumnya digunakan adalah kecepatan *read* dan *write* yang dihitung menggunakan satuan MB/s.
### Economy
*Economy* merupakan biaya relatif yang dikeluarkan untuk sebuah solusi berdasarkan manfaatnya. Penggunaan RAID yang terdiri dari beberapa perangkat *storage* berkapasitas kecil memliki kemungkinan lebih ekonomis dibanding membeli sebuah perangkat *storage* yang berkapasitas besar.

## Teknik Penyimpanan Data pada RAID

### Striping
Teknik *striping* memungkinkan data untuk didistribusikan ke beberapa *drive*. Hal tersebut dapat meningkatkan performa secara signifikan. Dikarenakan didistribusikan ke beberapa *drive*, kegagalan yang terjadi pada sebuah *drive* dapat mengakibatkan kehilangan seluruh data.
### Mirroring
*Mirroring* adalah teknik melakukan duplikasi data ke dua atau lebih *drive*. Teknik ini memiliki karakteristik *redundancy* sehingga ketika sebuah *drive* mengalami kegagalan tidak menyebabkan kehilangan data.
### Parity
*Parity* merupakan teknik *error checking* yang digunakan untuk mencegah kehilangan data, bekerja dengan cara menyimpan *checksums* secara terpisah dari datanya. *Checksum* adalah suatu nilai yang digunakan untuk mendeteksi kesalahan pada suatu data. Teknik *parity* memungkinkan blok data untuk direkonstruksi ketika terjadi kesalahan tanpa mengorbankan kecepatan dan kapasitas *drive*.
### Double Parity
Berbeda dengan *parity* yang hanya menggunakan satu informasi *parity*, *double parity* menggunakan dua informasi *parity* untuk melakukan proses *error checking*.
## RAID Levels
Terdapat beberapa cara untuk mengimplementasi RAID dengan menggunakan kombinasi berbagai teknik penyimpanan data. Beberapa cara tersebut umumnya dikenal dengan RAID *levels*. Perbadingan tiap level yang umum digunakan secara garis besar dapat dilihat pada tabel di bawah ini.
| RAID *Level* | Teknik Penyimpanan Data       | Minimal *Drives* yang Dibutuhkan | Kapasitas |
|:----------:|-------------------------------|-------------|-----------|
|      0     | *Striping*                      |      2      |     N     |
|      1     | *Mirroring*                     |      2      |    N/2    |
|      5     | *Striping* dengan *parity*       |      3      |    N-1    |
|      6     | *Striping* dengan *double parity* |      4      |    N-2    |
|   10(1+0)  | *Miroring* dan *striping*         |      4      |    N/2    |

Semua level RAID diatas menyimpan data di *drive* dalam bentuk *block*. Berikut merupakan penjelasan singkat beberapa level RAID yang umum digunakan:
### RAID 0
RAID 0 menggunakan teknik *striping* tanpa *redundancy*. Dikarenakan tidak memiliki *fault tolerance*, penggunaan RAID 0 sama sekali tidak direkomendasikan. Tujuan utama RAID 0 hanya untuk peningkatan performa dan pemakaian kapasitas yang maksimal. RAID 0 mulai ditinggalkan karena adanya SSDs (*solid-state drives*) yang harganya mulai terjangkau dan kapasitas yang semakin bertambah.

![Ilustrasi RAID 0 *Setup*](/assets/img/posts/RAID_0.svg.png)

Kelebihan:
- Meningkatkan performa serta RAID yang paling cepat.
- Biaya relatif tidak mahal.
- Kapasitas semua drive bisa terpakai secara maksimal.
- Mudah untuk melakukan *setup*.

Kekurangan:
- Jika sebuah *drive* mengalami kegagalan, maka akan mengakibatkan kehilangan seluruh data tanpa memiliki kesempatan untuk *recovery*.
### RAID 1
RAID 1 menggunakan teknik *mirroring*. Tiap *drive* memiliki salinan data yang sama persis di *drive* lainnya.

![Ilustrasi RAID 1 *Setup*](/assets/img/posts/RAID_1.svg.png)

Kelebihan:
- Memiliki *redundancy* yang tinggi.
- Jika sebuah *drive* mengalami kegagalan, sistem tetap dapat beroperasi tanpa kehilangan data.

Kekurangan:
- Kapasitas maksimal yang bisa terpakai hanya sebesar 50% dari jumlah *drive* yang ada.
- Tidak mengalami peningkatan performa. Jika terdapat perbedaan kecepatan antar *drive* yang digunakan di RAID 1, maka performa *write* secara keseluruhan adalah kecepatan dari *drive* yang paling lambat.
### RAID 5
RAID 5 menggunakan teknik *striping* dengan *parity*. Baik data maupun *parity* tersimpan secara terdistribusi di semua *drive* yang tersedia. Blok data dan informasi *parity*-nya tidak disimpan pada satu *drive* yang sama. Ketika terjadi kegagalan pada suatu *drive*, sistem akan membaca *parity* di semua *drive* yang masih tersedia untuk melakukan *rebuild* terhadap data yang hilang.

Proses *rebuild* meliputi penggantian *drive* yang mengalami kegagalan serta melakukan rekonstruksi terhadap blok data yang hilang ke *drive* yang baru. 

![Ilustrasi RAID 5 *Setup*](/assets/img/posts/RAID_5.svg.png)

Kelebihan:
- Meningkatkan performa.
- Sistem dapat bertahan ketika terdapat satu *drive* mengalami kegagalan dalam satu waktu.

Kekurangan:
- Jika terdapat dua *drive* mengalami kegagalan dalam waktu bersamaan, maka akan mengakibatkan kehilangan seluruh data.
-Proses *rebuild* ketika satu atau dua *drive* mengalami kegagalan membutuhkan waktu yang bergantung pada kapasitas *drive* dan jumlah data yang hilang.

### RAID 6
Hampir serupa dengan RAID 5, namun RAID 6 menggunakan teknik *striping* dengan *double parity*. RAID 6 meningkatkan *fault tolerance* dengan cara mampu mengatasi dua *drive* yang mengalami kegagalan.

![Ilustrasi RAID 6 *Setup*](/assets/img/posts/RAID_6.svg.png)

Kelebihan:
- Meningkatkan performa tapi performa *write* tidak lebih baik dibanding RAID 5.
- Sistem dapat bertahan ketika terdapat dua *drive* mengalami kegagalan dalam waktu bersamaan atau ketika satu *drive* mengalami kegagalan, lalu dilanjutkan dengan *drive* kedua mengalami kegagalan saat proses rekonstruksi data dari *drive* pertama yang gagal. Memiliki *fault tolerance* yang lebih tinggi dibanding RAID 5.

Kekurangan:
- Biaya relatif lebih mahal dibandingkan RAID 5.
- Proses *rebuild* ketika satu atau dua *drive* mengalami kegagalan membutuhkan waktu yang bergantung pada kapasitas *drive* dan jumlah data yang hilang.

### Nested RAID
*Nested* RAID merupakan RAID *levels* yang mengkombinasikan dua atau lebih level. Beberapa diantaranya adalah RAID 01, RAID 10, RAID 100, RAID 50 dan RAID 60. Sebagai contoh, akan dijelaskan secara singkat tentang salah satu *nested* RAID yang populer digunakan, yakni RAID 10.

### RAID 10 (1+0)
RAID 10 merupakan penggabungan teknik yang ada pada RAID 1 dan RAID 0. RAID 10 memiliki urutan proses yang berkebalikan dengan RAID 01. Pada saat melakukan *set up* RAID 10, pertama teknik yang digunakan adalah *mirroring*, selanjutnya adalah *striping*.

![Ilustrasi RAID 10 *Setup*](/assets/img/posts/480px-RAID_10_01.svg.png)

Kelebihan:
- Meningkatkan performa.
- Memiliki *redundancy* yang tinggi.
- Jika sebuah *drive* mengalami kegagalan, sistem tetap dapat beroperasi tanpa kehilangan data.

Kekurangan:
- Kapasitas maksimal yang bisa terpakai hanya sebesar 50% dari jumlah *drive* yang ada.







