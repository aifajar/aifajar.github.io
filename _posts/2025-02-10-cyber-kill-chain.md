---
title: Cyber Kill Chain
date: 2025-08-04 23:59:59 +0700
categories: [Cybersecurity]
tags: [cyber kill chain, security frameworks]
---

**Cyber Kill Chain** merupakan sebuah *framework* yang berisi tujuh tahap yang biasanya dilakukan oleh *threat actor* dalam melancarkan serangan siber. *Framework* ini dikembangkan oleh perusahaan pembuat senjata asal Amerika Serikat Lockheed Martin. Cyber Kill Chain sudah diadopsi secara global oleh komunitas *cybersecurity* untuk membantu para profesional mengidentifikasi dan mencegah serangan siber. Semakin awal tahap yang bisa diidentifikasi dan dicegah, semakin sedikit juga dampak yang ditimbulkan ke sistem yang dijadikan target oleh *threat actor*. 

*Framework* sangat berguna ketika kita ingin mengembangkan *incident response playbooks* serta memberikan informasi tambahan ketika kita melakukan investigasi terhadap suatu kasus serangan siber.

Terdapat tujuh tahap dalam *framework* Cyber Kill Chain, yaitu:
1. Recoinnassance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command and Control (C2)
7. Action on objectives.

## Recoinnassance
**Recoinnassance** merupakan proses mengumpulkan informasi terhadap target yang ingin dijadikan sasaran serangan siber. Semakin banyak informasi relevan yang dikumpulkan *threat actor* terkait targetnya, semakin bagus juga persiapannya dalam melancarkan serangan. Dalam proses ini terdapat dua metode pengumpulan informasi, yaitu dengan cara *passive* maupun dengan cara *active*.

Pengumpulan informasi cara *passive* dapat dilakukan dengan mengunjungi berbagai sumber *online* yang tersedia di internet, diantaranya adalah *search engine*, media sosial, forum *online*, web lowongan kerja sampai ke penggunaan OSINT. Beberapa informasi yang dapat dikumpulkan dengan cara ini, antara lain *stack* teknologi (infrastruktur dan aplikasi) serta informasi terkait detail karyawan.

Dengan cara *active*, biasanya *threat actor* mengumpulkan informasi secara langsung dari targetnya. Contohnya melakukan *scanning* terhadap laman web resmi dari perusahaan target atau dari nama domain (DNS). Beberapa informasi yang dapat dikumpulkan dengan cara ini, antara lain *IP address*, *open ports*, subdomain dan direktori atau URL tersembunyi.

## Weaponization
Setelah melakukan tahap *recoinnassance* biasanya *threat actor* akan membuat program berbahaya berupa *tool* atau *malware* untuk mengeksploitasi kerentanan yang ada pada sistem target. Ketika program berbahaya berhasil dijalankan maka akan berdampak terhadap *confidentiality*, *integrity*, atau, *availability* pada sistem target. Umumnya program berbahaya yang dibuat juga berisi *backdoor* yang memungkinkan *threat actor* untuk mengakses sistem target secara *remote*.

## Delivery
Pada tahap ini *threat actor* akan mengirimkan program berbahaya berupa *tool* atau *malware* yang sudah dibuat ke target dengan harapan target akan mengeksekusi program berbahaya tersebut. Biasanya medium penyebaran yang digunakan adalah email, USB dan laman web. 

## Exploitation
*Threat actor* akan mengeksploitasi kerentanan (vulnerabilities) yang ada pada sistem target. Beberapa contoh aktivitas pada tahap ini adalah mengeksploitasi kerentanan pada *software* (tidak adanya *input validation*, *hard-code credentials*), server (*open ports insecure protocols*, *weak configuration*, *priviledge access*) maupun *human errors* pada sistem target.

## Installation
Pada tahap ini *threat actor* akan melakukan instalasi *backdoor* agar bisa mengakses sistem target yang terinfeksi kapan saja. Penting bagi *threat actor* agar membuat *backdoor*-nya *persistence* dengan harapan *backdoor* masih dapat diakses walaupun sistem target di-*reboot*. 

Setelah instalasi *backdoor*, biasanya akan ada instalasi lanjutan untuk program berbahaya lainnya. Dalam melakukan aksinya, *threat actor* perlu memastikan agar program berbahaya yang terpasang tidak terdeteksi oleh target.

## Command and Control (C2)
*Command and Control* (C2) merupakan tahap dimana *threat actor* berhasil membuat koneksi antara sistem target dengan mesin/komputer miliknya secara *remote*. Koneksi tersebut memungkinkan *threat actor* untuk menjalankan instruksi (*command*), mengambil data serta mengontrol interaksi eksternal di sistem target. Tahap C2 dapat menjadi jembatan untuk menginfeksi lebih banyak *resources* di sistem target.

Saat ini, banyak *threat actor* yang sudah membuat *traffic* yang sah menggunakan protokol HTTPS dan DNS maupun teknik *tunneling* pada koneksi *remote* agar terhindar dari deteksi.

## Action on objectives
*Action on objectives* merupakan tahap akhir dari Cyber Kill Chain. Pada tahap ini, *threat actor* akan fokus ke tujuan utama dari dilancarkannya serangan siber ke sistem target. Beberapa tujuan utama *threat actor*, antara lain mencuri data sensitif dan menjualnya ke *dark web* atau melancarkan serangan DDoS.

## Contoh Aktivitas Setiap Tahap

| No. | Tahap | Contoh Aktivitas |
|---|---|---|
| 1 | *Recoinnassace* | Mengumpulkan informasi target melalui laman web, media sosial, dan OSINT. |
| 2 | *Weaponization* | Membuat *malware*. |
| 3 | *Delivery* | Menyebarkan *malware* melalui USB. |
| 4 | *Exploitation* | Mengeksploitasi *vulnerabilities* pada sistem, seperti kredensial yang bocor dan terbukanya *port* untuk insecure protocol* ke publik. |
| 5 | *Installation* | Menginstall *malware* di server target. |
| 6 | *Command and Control* (C2) | Menjalankan botnet pada beberapa server target yang sudah terinfeksi. |
| 7 | *Action on objectives* | *Threat actor* berhasil mencuri data. |