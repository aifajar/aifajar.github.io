---
title: Model Layanan pada Cloud
date: 2024-01-02 23:59:59 +0700
categories: [Cloud Computing]
tags: [cloud service, iaas, paas, saas]
---

Model layanan atau servis pada *public cloud* umumnya terbagi menjadi 3, yaitu IaaS, PaaS dan SaaS. Apapun penyedia layanan *cloud*-nya (*cloud servics provider* atau lebih dikenal dengan singkatan CSP), baik AWS, Azure, atau Google Cloud, semua layanan produk mereka secara garis besar akan terbagi menjadi 3 model tersebut.

## Infrastructure as a Service (IaaS)
IaaS menawarkan fleksibilitas dan kontrol mayoritas IT *resources* yang yang disediakan penyedia ;ayanan *cloud* kepada pelanggan (*customers*). Penyedia layanan *cloud* memungkinkan pelanggan untuk memilih tipe *resources* dan konfigurasi infrastruktur IT yang dibutuhkan. Cara kerja IaaS menggunakan konsep virtualisasi. Virtualisasi membuat pelanggan dapat menjalankan aplikasi atau sistem IT di perangkat *virtual* sama seperti mereka menjalankannya menggunakan perangkat fisik.

Berikut merupakan 3 kategori pembagian *resources* pada IaaS beserta contoh pendukungnya.
1. *Compute*: CPU, GPU, RAM.
2. *Storage*: SSD, HDD, NAS, Object Storage.
3. *Network*: VPC networks, Firewall, Load Balancer.

Contoh layanan dari IaaS adalah Google Cloud Compute Engine, AWS EC2, dan AWS S3.

## Platform as a Service (PaaS)
PaaS memungkinkan pelanggan untuk fokus kepada pengembangan aplikasi tanpa memikirkan beberapa *layer* pendukung infrastruktur, seperti sistem operasi yang dibutuhkan agar aplikasi dapat berjalan. Ketika menggunakan PaaS, kita tidak perlu memikirkan beberapa *tasks* seperti perencanaan kapasitas *resources*, *server maintenance*, *patching*, dsb.

Contoh layanan dari PaaS adalah Azure SQL Database dan Google Kubernetes Engine.

## Software as a Service (SaaS)
*Public cloud* menawarkan sebuah produk berupa aplikasi kepada pelanggan. Pelanggan tidak perlu merasa khawatir untuk memikirkan detail implementasi dari infrastruktur IT yang membuat aplikasi tersebut berjalan. Aplikasi biasanya sudah bisa tersedia ke publik melalui internet dan pelanggan hanya perlu menggunakannya saja berdasarkan jenis pembelian.

Contoh dari SaaS adalah Microsoft Office 365 dan Google Drive.

## Anything as a Service
Selain ketiga model di atas, terdapat juga istilah "*as a service*" pada banyak servis. Beberapa diantaranya adalah DaaS (Desktop as a Service), FaaS (Function as a Service), SecaaS (Security as a Service), dan masih banyak lagi. Istilah-istilah *as a service* tersebut biasanya ditujukan untuk meningkatkan nilai *marketing* dari suatu produk.

## Layers pada Infrastruktur Cloud
Infrastuktur IT pada *cloud* memiliki beberapa *layers* pendukung. Berikut merupakan beberapa *layers* pada infrastruktur IT *cloud* beserta fungsinya.

| *Layer* | Fungsi |
|---|---|
| Data | Data yang disimpan, dikelola maupun yang digunakan oleh*application*. |
| *Application* | Produk atau servis yang berupa program yang dijalankan oleh pelanggan serta disediakan oleh *cloud provider*. |
| *Runtime* | Komponen *software* yang dibutuhkan untuk menjalankan *application*. |
| *Middleware* | Perantara antara sistem operasi dan *application* yang berjalan. |
| Sistem Operasi | *Software* dasar yang digunakan untuk menjalankan server virtual. |
| *Hypervisor* | *Software* yang berjalan di server fisik untuk memungkinkan server virtual atau VMs (Virtual Machines) dapat berjalan serta berbagi *resources* yang dimiliki oleh server fisik. |
| Server Fisik | Komputer atau mesin fisik yang menjadi *host* bagi server virtual. |
| Perangkat *Storage* | Sistem *hardware* yang menyediakan penyimpanan data pada server fisik. |
| Perangkat *Network* | Perangkat fisik terkait *network* seperti *switches* dan *routers* untuk menghubungkan server fisik ke internet. |

## Shared Responsibility
Jika kita menggunakan infrastruktur IT *on-premise*, kita akan bertanggung jawab terhadap semua aspek dari suatu produk atau layanan yang kita tawarkan, mulai dari perangkat fisik *network*, *hardware*, sistem operasi, aplikasi, sampai ke data. Namun, jika kita menggunakan layanan dari *public cloud computing* berdasarkan ketiga jenis model di atas, penyedia layanan *cloud* dan pelanggan memiliki *shared responsibility* terhadap pengelolaan *layers* dari infrastruktur *cloud* yang ada. Pengelolaan *layers* dapat berupa pemilihan dan penggunaan *stack technology* tertentu maupun pengaturan terkait aspek keamanannya (*security*). Berikut merupakan ilustrasi *shared responsibility* antara penyedia layanan *cloud* dan pelanggan.

![img-description](/assets/img/posts/cloud-computing/shared-responsibility.png)
_Ilustrasi Shared Responsibility pada Servis Cloud_