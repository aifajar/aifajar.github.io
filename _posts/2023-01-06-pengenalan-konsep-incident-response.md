---
title: Pengenalan Konsep Incident Response
date: 2023-01-06 23:59:59 +0700
categories: [Cybersecurity, CompTIA Security+]
tags: [cybersecurity,comptia security+,incident response]
---

Sebelum mendalami lebih lanjut tentang konsep dan pengertian *incident response*, ada baiknya untuk mengetahui istilah-istilah terkait dengan *incident* atau insiden.

- *Event*: Setiap peristiwa atau kejadian yang dapat diamati dalam sistem.
- *Incident*: Suatu *event* yang memberikan dampak negatif karena melanggar *confidentiality*, *integrity*, atau *availability* selama masa data diproses, disimpan atau ditransmisikan.
- *Breach*: Kondisi dimana orang yang tidak berwewenang atau sah (*unauthorized user*) memiliki akses terhadap suatu sistem (meliputi komputer, jaringan, situs web, *database* dan sebagainya) serta juga dapat terjadi ketika orang yang berwewenang (*authorized user*) namun dapat melakukan akses selain yang telah ditetapkan. Beberapa dampak yang dihasilkan oleh *breach* antara lain adalah kehilangan kontrol suatu sistem dan akses terhadap data rahasia.
- *Intrusion*: Sebuah *security event* atau kombinasi beberapa *event* yang menghasilkan insiden *security* dimana seorang penyusup berusaha untuk mengakses sumber daya sistem tanpa otorisasi (*authorization*). 

*Incident response* merupakan suatu prosedur yang diikuti oleh suatu organisasi ketika mereka mengalami suatu insiden. Tujuan *incident response* adalah untuk meminimalisir dampak yang ditimbulkan akibat suatu insiden serta memastikan bahwa organisasi dapat kembali beroperasi secepatnya. Prosedur yang ditetapkan suatu organisasi biasanya tertuang pada sebuah *policy* ataupun *incident response plan* yang telah mereka buat. 

*Policy* merupakan pedoman untuk segala aktivitas organisasi agar sesuai dengan standar industri dan patuh terhadap regulasi dan hukum yang ada. *Policy* diterapkan ke suatu organisasi dalam berbagai tingkatan, tergantung jabatan seluruh anggotanya. *Incident response plan* (IRP) merupakan pedoman untuk merespons suatu insiden agar meminimalisir dampak yang ditimbulkan terhadap organisasi. IRP dapat berisi daftar orang-orang yang bertanggung jawab untuk merespons, sumber daya yang tersedia serta langkah yang akan diambil ketika insiden sedang terjadi maupun sesudah diselesaikan.

## Siklus Incident Response

Siklus *incident response* memiliki beberapa fase. Terdapat perbedaan istilah di fase yang ada pada posting ini dan yang terdapat pada *exam objective* sertifikasi CompTIA Security+. Posting ini merujuk kepada [NIST Computer Security Incident Handling Guide special publication](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf). Fase *detection & analysis* merujuk kepada *identification* di CompTIA. Secara garis besar fase yang dilakukan pada keduanya ini sama, namun hanya terdapat perbedaan istilah saja.

![Siklus Incident Response](/assets/img/posts/3/SiklusIncidentResponse.PNG)

Berikut merupakan penjelasan singkat dari tiap fase yang terdapat pada siklus *incident response*.

### Preparation
*Preparation* merupakan usaha ataupun aktivitas yang dilakukan untuk mencegah terjadinya insiden. Selain mencegah, fase ini juga mempersiapkan *incident response* secara matang ketika insiden terjadi. Fase ini haruslah tetap *up-to-date* agar tetap relevan dengan perkembangan dunia *cybersecurity*. Beberapa aktivitas umum yang dapat dilakukan antara lain:
- Membuat *policy* ataupun IRT sebagai pedoman *incident response* organisasi.
- Melakukan pelatihan *security awareness* terhadap anggota teknis maupun nonteknis.
- Membentuk tim *incident response*.
- Merencanakan koordinasi komunikasi antar *stakeholders*.
- Melakukan *hardening* pada sistem yang ada. *Hardening* merupakan proses penggunaan berbagai solusi (*tools*, teknik, *best practice*, dan sebagainya) untuk memperkuat keamanan suatu sistem dan mengurangi *vulnerabilities* yang ada, contoh umumnya melakukan *patching* dan *update* sistem operasi secara berkala. 

### Detection & Analysis
Fase ini terdiri dari dua proses utama, yakni deteksi (*detection*) dan analisis (*analysis*). Proses *detection* dapat dilakukan dengan cara memonitor setiap *event* yang terjadi pada sistem. Jika suatu *event* menghasilkan *alert*, maka akan dilanjutkan ke proses analisis untuk menentukan apakah *alert* yang dihasilkan merupakan *true positive* atau *false positive*. *False positive* adalah kondisi dimana sistem mendeteksi suatu ancaman (*threat*) yang ditandai dengan adanya *alert*, namun yang terjadi sebenarnya adalah tidak ada ancaman.

### Containment, Eradication & Recovery
*Containment* merupakan proses membatasi ruang lingkup insiden yang terjadi sehingga bisa meminimalisir dampak yang ditimbulkan. Salah satu cara umum yang digunakan pada proses *containment* adalah melakukan isolasi atau karantina terhadap komponen yang terkena dampak insiden. Pengumpulan bukti (*evidence*) dapat dilakukan pada proses ini. *Eradication* merupakan proses menghilangkan penyebab suatu insiden terjadi, contohnya menghapus *malware* dan menonaktifkan akun yang terkena *breach*.

*Recovery* merupakan proses mengembalikan sistem kembali beroperasi dan berfungsi secara normal. Contoh proses *recovery* adalah melakukan *restore* kembali ke *secure state*(kondisi sistem yang aman, sebelum terjadinya insiden) menggunakan *backup*. Setelah melakukan proses *recovery*, sistem akan dimonitor dalam waktu tertentu untuk mendeteksi dan mencegah insiden yang sama terjadi lagi. Proses iterasi dari *recovery* kembali ke fase deteksi dan analisis lalu ke proses *containment* dan *eradication* mungkin akan terjadi berulang kali untuk menghasilkan resolusi yang baik.

### Post-Incident Activity
Pada fase ini terdapat proses *lesson learned*, yakni proses belajar dan menentukan peningkatan apa yang harus dilakukan. Selain proses tersebut, penting untuk melakukan dokumentasi secara lengkap tentang insiden yang terjadi. Fase ini akan berisi *feedback* untuk fase *preparation* selanjutnya.

