---
title: Loki - DevOps Tool untuk Sistem Logging
date: 2025-01-27 23:59:59 +0700
categories: [DevOps]
tags: [devops, devops tools, logging system, loki]
---

![Logo](/assets/img/posts/logo/logo-loki.svg)

Loki merupakan sebuah *tool* DevOps dari Grafana yang digunakan terkait sistem *logging*. Fungsi utamanya adalah sebagai sistem *log aggregation* yang menyimpan log dan melakukan proses *query* terhadap log-log tersebut. **Log aggregation** merupakan proses pengumpulan banyak log dari berbagai sumber yang ada dan menggabungkannya ke dalam satu tempat penyimpanan yang tersentralisasi. *Log aggregation* memudahkan sebuah tim untuk memantau, mengecek atau menganalisa suatu sistem, misalnya sistem operasi, database, dan aplikasi. *Log aggregation* dapat digunakan juga sebagai persyaratan suatu regulasi maupun *compliance*. Beberapa *tools* terkait *logging* populer selain Loki yang digunakan dalam dunia industri, antara lain ELK (Elasticsearch, Logstash, Kibana) Stack, Datadog, New Relic, dan Splunk.

## Konsep Sistem Logging

Sistem *logging* harus mampu mengumpulkan log dari berbagai sumber yang ada. Setelah log dikumpulkan, maka log-log tersebut akan disimpan secara terpusat. Penting untuk diingat bahwa, salah satu *concern* paling penting dari sistem *logging* adalah banyaknya volume log yang harus ditampung. Hal tersebut dapat berdampak ke aspek biaya maupun performa. Sistem *logging* yang baik harus mendukung skalabilitas untuk mengatasi volume log yang besar.

Pada sistem *logging* biasanya log akan diindeks. Sama seperti di *database*, log diindeks untuk melakukan proses pencarian dengan cepat. Log juga harus diatur masa retensinya. Masa retensi tiap log bisa berbeda-beda tergantung jenisnya, contohnya log terkait keamanan atau audit biasanya disimpan lebih lama dibanding log terkait aplikasi untuk menganalisis suatu isu.

Beberapa kegunaan sistem *logging* diantaranya adalah dapat memudahkan kita untuk mencari suatu penyebab ataupun *root cause* dari suatu isu yang terjadi di sistem aplikasi atau infrastruktur secara terpusat. Log juga dapat menyediakan konteks tambahan ketika kita ingin saat menganalisis isu secara lebih mendalam, contohnya mengaitkan proses *spike* pada CPU dengan log *request* tertentu.

## Fitur Utama Loki

Dibandingkan dengan sistem *logging* yang lain, penggunaan Loki bisa dibilang sangat naik daun dalam beberapa tahun belakangan. Selain karena *open source*, Loki didesain untuk membuat biaya operasional jadi lebih efisien serta mudah untuk digunakan. Salah satu fitur yang patut di-*highlight* dari Loki dibanding dengan *tools* lainnya adalah Loki tidak mengindeks konten dari log, namun hanya mengindeks metadata (seperti label) dari log. Hal tersebut yang membuat Loki menjadi *tool* yang lebih efisien dari segi biaya maupun performa.

Log yang dikirim ke Loki mendukung berbagai format serta berbagai sumber penghasil log. Loki juga mendukung penyimpanan log menggunakan *object storage* sehingga skalabilitas terkait volume log yang sangat besar dapat terjamin serta dukungan biaya yang efektif dan penyimpanan yang *durable* dari layanan *object storage*.

Konfigurasi dan sintaksis *query* yang digunakan Loki sangat mirip dengan Prometheus. Prometheus sendiri merupakan inspirasi dalam pembuatan Loki, Prometheus sebagai *tool* pengumpulan *metrics* sedangkan Loki sebagai *tool* pengumpulan log. Loki dan Prometheus biasanya digunakan bersama dalam suatu *stack observability*. Loki mudah dintergrasikan ke Kubernetes maupun ke Grafana untuk melakukan visualisasi dari log yang ada.

## Komponen Pendukung Loki

Untuk mengumpulkan log dari berbagai sumber, Loki membutuhkan sebuah *log agent*. *Log agent* akan diinstalasi di komputer atau sistem klien (target). Beberapa *agent* pada sistem klien yang dapat digunakan Loki, antara lain Promtail, Fluentd, dan Logstash. Namun dalam tulisan selanjutnya, saya hanya akan membahas terkait Promtail saja. Berikut merupakan beberapa fungsi Promtail sebagai *log agent* pada sistem *client* di Loki.
1. *Log collector*: Promtail mengambil log dari berbagai sumber yang ada di sistem klien, misalnya log dari aplikasi atau infratruktur, seperti *database*.
2. Menambahkan label: Promtail akan menambahkan label pada log yang telah dikumpulkan. Label-label tersebut akan sangat berguna untuk memfilter dan mengelompokkan log ketika akan dianalisi di Loki. Contoh dari label berupa app_name, environment, atau hostname.
3. *Parsing log*: Promtail dapat memecah atau memformat log sesuai kebutuhan analisis dari pengguna. *Parsing* akan menghasilkan label baru pada log.
4. Mengirim log ke Loki: Setelah log dikumpulkan, dilabeli dan di-*parsing*, Promtail akan mengirimkannya ke Loki untuk disimpan.

Loki menggunakan LogQL sebagai bahasa *query*-nya. LogQL dapat digunakan di Grafana secara langsung untuk melakukan visualisasi log. Jika terbiasa dengan menggunakan *command line*, kita juga dapat menggunakan LogCLI untuk melakukan explorasi *query* pada log. Selain itu, Loki juga mendukung sistem *alerting* pada log. *Alert rules* dapat dibuat menggunakan Prometheus Alertmanager.

## Projects Terkait Loki

Berikut merupakan beberapa *projects* terkait Loki yang saya buat dokumentasinya.

1. [Instalasi dan Pengaturan Loki & Promtail](https://blog.aifajar.com/posts/instalasi-dan-pengaturan-loki-promtail/)
2. [Visualisasi Log Nginx (Access & Error Logs) Menggunakan Loki](https://blog.aifajar.com/posts/visualisasi-log-nginx-access-error-logs-menggunakan-loki/)
3. [Visualisasi dan Analisis Log Slow Query MongoDB Menggunakan Loki](https://blog.aifajar.com/posts/visualisasi-dan-analisis-log-slow-query-mongodb-menggunakan-loki/)

> Saya sendiri melakukan implementasi *projects* tersebut menggunakan infrastruktur yang sudah ada (*existing infrastructure*), sehingga tidak menggunakan versi terbaru dari *stack* aplikasi atau *tools* yang ada. Walaupun demikian, saya yakin bahwa secara implementasi dan konsep tidak akan berbeda jika menerapkannya menggunakan versi terbaru dari *stack* aplikasi atau *tools* ataupun menggunakan aplikasi atau *tools* berbeda namun memiliki kegunaan yang sama.