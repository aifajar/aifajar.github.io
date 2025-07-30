---
title: Autentikasi "Something You Are" - Biometrik
date: 2025-07-31 23:59:59 +0700
categories: [Cybersecurity]
tags: [cybersecurity, autentikasi, comptia sec+]
---

Salah satu metode autentikasi pada suatu sistem yang dapat digunakan adalah dengan menggunakan biometrik. Biometrik adalah karakteristik seseorang yang berupa fisik ataupun perilaku yang biasanya bersifat unik. Terdapat beberapa keunggulan ketika menggunakan biometrik sebagai metode autentikasi, antara lain sulit dipalsukan dan nyaman digunakan (tanpa perlu mengingat atau mempunyai sesuatu). Walaupun memiliki beberapa keunggulan, bukan berarti metode tersebut tidak memiliki kelemahan. Kelemahan terkait kesalahan atau eror pada autentikasi akan dibahas lebih lanjut. Topik ini termasuk ke dalam salah satu *exam objective* pada sertifikasi CompTIA Security+: SY0-601.

## Autentikasi "Something You Are"

*Something you are* adalah salah satu faktor yang ada dalam autentikasi selain *something you have* dan *something you know*. Biasanya bisa lebih dari satu faktor digunakan untuk meningkatkan keamanan suatu sistem. Autentikasi *something you are* mengandalkan identitas biologis yang melekat pada individu.

## Jenis Biometrik

Di bawah ini merupakan jenis biometrik yang dapat dijadikan sebagai metode autentikasi.

### Fingerprint Scanner

Sidik jari atau *fingerprint* setiap orang berbeda-beda. **Scanner** merupakan perangkat fisik yang digunakan untuk mengindentifikasi beberapa jenis biometrik. Salah satu penerapan umum jenis biometrik ini adalah  membuka *smartphone* menggunakan *fingerprint*. Contoh lain penerapan jenis biometrik ini adalah saat melakukan perekaman Kartu Tanda Penduduk Elektronik (e-KTP) bagi warga Indonesia yang sudah cukup umur. Setia warga akan diminta untuk merekam sidik jari lalu data sidik jarinya akan tersimpan di *database* kependudukan nasional. Data sidik jari ini akan digunakan selanjutnya untuk verifikasi identitas saat mengurus dokumen, contohnya saat pembuatan rekening bank.

![contoh-gambar-fingerprint](/assets/img/posts/cybersecurity/contoh-gambar-fingerprint-scanner.jpg)
_Contoh Perangkat Fingerprint Scanner. Sumber: https://www.cardlogix.com/product/futronic-fs80h/_


### Retina Scanner

*Retina scanner* adalah perangkat autentikasi biometrik yang menganalisis pola pembuluh darah pada retina seseorang. Retina adalah lapisan tipis di bagian belakang mata yang peka terhadap cahaya. Setiap orang memiliki pola pembuluh darah yang unik pada retina mereka.

### Iris Scanner

Iris adalah bagian mata yang berwarna yang mengelilingi pupil. Setiap orang memiliki pola iris yang unik, dan biasanya pola ini tetap stabil seumur hidup.

### Voice Recognition

*Voice recognition* merupakan autentikasi yang digunakan untuk mengidentifikasi atau memverifikasi identitas seseorang berdasarkan pola suara mereka.
Pola suara dapat dianalisis dari frekuensi, intonasi, gaya bicara, dan struktur fisiologis saluran suara. Jenis ini cocok untuk penggunaan jarak jauh atau sistem yang berbasis audio, namun biasanya rentan terhadap gangguan lingkungan.

### Facial Recognition

*Facial recognition* menggunakan ciri-ciri unik wajah individu. Autentikasi ini biasanya menganalisis struktur geometri dan titik-titik kunci (*landmarks*) pada wajah seperti jarak antar mata, bentuk rahang, kontur bibir, dan ciri lainnya. Beberapa contoh penerapan *facial recognition* yang sudah umum, antara lain autentikasi pada *smartphone* (Face ID di iPhone) dan sistem kehadiran atau absensi.

![contoh-gambar-facial-recognition](/assets/img/posts/cybersecurity/contoh-gambar-facial-recognition.jpg)
_Contoh Facial Recognition. Sumber: https://www.macrumors.com/2017/09/20/facial-recognition-startups-face-id/_

### Vein

Jenis biometrik ini menggunakan pola pembuluh darah (vena) di telapak tangan menggunakan sinar inframerah.

### Gait Analysis

*Gait Analysis* adalah proses menganalisis pola berjalan seseorang untuk mengidentifikasi dan memverifikasi identitasnya. Beberapa hal yang dianalisis terkait pola berjalan seseorang, antara lain panjang langkah, gerakan sendi (pinggul, lutut), kecepatan berjalan, dan postur tubuh. *Gait analysis* termasuk ke biometrik perilaku. Metode autentikasi jenis biometrik ini dapat digunakan dari kejauhan tanpa kontak fisik.

## Eror pada Biometrik

Walaupun bimetrik bersifat unik, bukan berarti dalam penerapannya tidak lepas dari kesalahan. Berikut beberapa hal yang dapat menyebabkan kesalahan autentikasi pada biometrik:

1. Pencahayaan yang tidak memadai

    Dapat berdampak pada tipe biometrik *facial recognition*, *retina*, atau *iris scanner*.

2. Perubahan karakteristik fisik.

    Perubahan yang disebabkan oleh cedera, pemakaian kacamata, atau pertambahan usia dapat menyebabkan kegagalan autentikasi dikarenakan data tidak sesuai dengan data yang terekam sebelumnya.

3. Perangkat atau sensor yang tidak akurat.

    Kamera berkualitas rendah atau sensor retina yang tidak sensitif dapat menyebabkan kesalahan dalam pembacaan data biometrik.

4. Konfigurasi sistem yang tidak optimal.

    Toleransi sistem yang terlalu longgar atau ketat dapat menyebabkan diterimanya pengguna yang tidak sah atau tertolaknya pengguna yang sah.

Kesalahan pada autentikasi biometrik umumnya terbagi menjadi dua, yaitu:

### False Acceptance Rate (FAR)

*False Acceptance Rate* (FAR) adalah persentase pengguna yang tidak sah namun diterima oleh sistem autentikasi. FAR termasuk ke dalam jenis **Type II Error**, dan secara statistik masuk ke dalam **False Positive**. FAR akan menurun seiring toleransi atau *threshold* yang meningkat.

### False Rejection Rate (FRR)

*False Rejection Rate* (FRR) adalah persentase pengguna yang sah namun tertolak oleh sistem autentikasi. FRR termasuk ke dalam jenis **Type I Error**, dan secara statistik masuk ke dalam **False Negative**. FRR akan meningkat seiring toleransi atau *threshold* yang meningkat pula.

### Crossover Error Rate (CER)

*Crossover Error Rate* (CER) adalah titik pertemuan antara FAR dan FRR. CER terjadi saat nilai FAR dan FRR sama besarnya. Sistem autentikasi biometrik dengan indikator baik adalah yang memiliki nilai CER yang rendah. Berikut merupakan ilustrasi grafik dari CER.

![grafik-CER](/assets/img/posts/cybersecurity/grafik_CER.PNG)
_Grafik Crossover Error Rate (CER)_