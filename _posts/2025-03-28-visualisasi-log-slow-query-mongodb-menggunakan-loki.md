---
title: Visualisasi Log Slow Query MongoDB menggunakan Loki
date: 2025-03-28 23:59:59 +0700
categories: [DevOps]
tags: [devops, devops tools, logging system, mongodb, promtail]
---

Sebelum mengikuti instruksi di bawah pastikan *tool* Promtail sudah terinstal di server target yang terdapat MongoDB-nya. Jika belum memiliki Promtail, silakan Anda cek tulisan saya sebelumnya [disini](https://blog.aifajar.com/posts/instalasi-dan-pengaturan-loki-promtail/) untuk mengikuti panduan instalasinya.

## Pengaturan Servis MongoDB

Secara *default*, setiap *query* yang dieksekusi dan memiliki durasi lebih dari 100 ms (0,1 detik) akan tercatat di log servis MongoDB. Dalam kasus ini, saya akan mengatur agar MongoDB hanya mencatat *slow query* yang memiliki durasi lebih dari 0,5 detik (500 ms) saja. Penentuan batas durasi yang dapat ditolerir, diatur melalui nilai *threshold* pada *file* pengaturan servis MongoDB. Lokasi *file* pengaturan tersebut berada di /etc/mongod.conf. Di bawah ini merupakan langkah untuk mengubah nilai *threshold* durasi pada pencatatan log *slow query*.

1. Buka *file* pengaturan servis MongoDB.

    ```console
    sudo nano /etc/mongod.conf
    ```

2. Lalu ubah pengaturan nilai *threshold* di *file* tersebut. Nilainya diwakili oleh atribut **slowOpThresholdMs**, posisinya berada di bawah pengaturan **operationProfiling**.

    ```console
    ...
    operationProfiling:
      mode: slowOp
      slowOpThresholdMs: 500
    ...
    ```

3. Setelah diubah, silakan *restart* servis MongoDB.

    ```console
    sudo systemctl restart mongod.service
    ```

Terdapat beberapa pertimbangan kenapa perlu mengubah nilai *default threshold*. Diantaranya adalah agar log hanya mencatat *queries* yang benar-benar lambat sesuai dengan toleransi durasi yang sudah ditentukan. Hal ini bertujuan agar server target tidak menciptakan terlalu banyak log, sehingga mengakibatkan kapasitas penyimpanan diska di server target maupun *object storage* yang digunakan Loki menjadi besar. Log juga dapat meningkatkan penggunaan *write* I/O *disk* di server. Peningkatan penggunaan I/O *disk* tanpa diimbangi dengan spesifikasi diska dan *hardware resources* yang mumpuni dapat mengakibatkan penurunan performa kinerja server. 

## Struktur Log Slow Query

Struktur log *slow query* pada MongoDB memiliki format JSON. Secara *default*, lokasi log servis MongoDB di server Ubuntu berada di /var/log/mongodb/mongod.log. Berikut merupakan contoh penampakan log *fake slow query* yang terdapat di MongoDB.

![log-slow-query](/assets/img/posts/devops/contoh-log-slow-query.png)
_Contoh Log Slow Query_

Struktur log dalam bentuk *clean version* tersebut kira-kira seperti ini.

```console
object {7}
  t {1}
    $date           : 2025-03-13T13:30:54.339+07:00
  s : I
  c : COMMAND
  id  : ...
  ctx : ...
  msg : Slow query
  attr {16}
    type            : command
    ns              : school.Student
    command {6}
        aggregate       : Student
        pipeline [6]
          ...
        allowDiskUse    : true
        cursor {0}
          ...
        lsid {1}
          $db           : school
    planSummary     : IXSCAN { sc.id: 1, sc.desc: 1 }, IXSCAN { sc.id: 1, cl.name: 1 }
    keysExamined    : 127037
    docsExamined    : 137038
    cursorExhausted : ...
    numYields       : ...
    nreturned       : 8
    queryHash       : ...
    planCacheKey    : ...
    reslen          : ...
    locks {6}
      ...
    storage {0}
      ...
    protocol        : ...
    durationMillis  : 5342
```

Menguraikan struktur log berformat JSON di atas dengan memecahnya menjadi *clean version* dapat membantu kita untuk memahami struktur log dan menentukan bagian mana saja yang penting dari log tersebut untuk di-*parsing*. Tidak semua bagian log penting untuk dikirim ke sistem *logging* terpusat. *Parsing* log akan membentuk label pada log sehingga kita dapat menggunakan label itu untuk melakukan kueri pada log. Kueri tersebut ditujukan untuk mengekstrak atau mengolah informasi penting pada log.

> Silakan gunakan [*tool*](https://codebeautify.org/jsonviewer) ini untuk memberikan gambaran struktur log secara jelas.

## Analisis Singkat Log Slow Query

Beberapa bagian dari log *slow query* di atas akan dijadikan sebagai label dan kemudian dikirim ke *tool* Loki. Berikut merupakan sedikit penjelasan dari beberapa label yang diperlukan dalam proses analisis atau *profiling slow query*.

- **t.$date**: *Timestamp* waktu dimulainya proses *query* berjalan.

- **msg**: Pesan yang menunjukkan konteks dari isi log. Jika berisi "Slow query" maka log tersebut merupakan pencatatan informasi terkait eksekusi *query* yang memiliki durasi lebih dari *threshold* yang sudah ditentukan.

- **attr.ns**: Lokasi *database* dan *collection* *query* dieksekusi.

- **attr.command**: Perintah lengkap dari proses suatu *query*, termasuk semua parameternya. Label ini bisa membantu untuk proses identifikasi suatu *query* apakah sudah ditulis secara optimal dan efisien, baik dari segi model data, operasi, indeks, parameter, dsb. Jika ingin mengambil informasi *query* berupa *pipeline* atau *find*, tanpa keseluruhan perintah untuk dijadikan sebagai label, maka dapat menggunakan **attr.command.pipeline** atau **attr.command.find**.

- **attr.planSummary**: Label yang menunjukkan bagaimana MongoDB menjalankan *query*. Umumnya terdapat dua cara menjalankan *query*, yaitu dengan menggunakan indeks (IXSCAN) atau tanpa menggunakan indeks (COLLSCAN). Menjalankan *query* tanpa menggunakan indeks dapat memperlambat proses eksekusi *query* yang bersifat *reads*. Namun, perlu diperhatikan juga jika ingin menambah indeks baru, penggunaan indeks secara berlebihan dapat menyebabkan proses eksekusi yang bersifat *writes* (*create* & *update*) menjadi lebih lambat serta membuat penggunaan penyimpanan diska dan memori di server tempat servis MongoDB berada menjadi lebih besar.

- **attr.docsExamined**: Jumlah *documents* yang diperiksa saat mengeksekusi suatu *query*. Semakin tinggi nilai label ini, maka besar kemungkinan *query* yang dieksekusi tidak efisien. Salah satu cara untuk mengurangi jumlah pemeriksaan *documents* saat mengeksekusi suatu *query* adalah dengan menerapkan indeks. Jika indeks sudah diterapkan tapi jumlah *documents* tetap tinggi, maka ada kemungkinan indeks tidak diterapkan secara baik. Salah satu strategi yang dapat diterapkan dalam penentuan indeks adalah dengan menggunakan indeks komposit (indeks pada lebih dari satu *field* di sebuah *collection*) yang menerapkan prinsip ESR (*Equality*, *Sort*, *Range Index*).

- **attr.nreturned**: Jumlah *documents* yang dikembalikan suatu *query*. Label ini biasanya digunakan untuk menganalisis performa *query* dengan cara membandingkannya dengan label **attr.docsExamined**. Jika nilai label **attr.nreturned** jauh lebih kecil dari label **attr.docsExamined**, maka besar kemungkinan *query* yang dieksekusi tidak efisien.

- **attr.keysExamined**: Jumlah indeks yang diperiksa saat mengeksekusi suatu *query*. Jika nilainya 0, maka *query* tidak menggunakan indeks. Kondisi ideal dari label ini adalah memiliki nilai yang mendekati dengan nilai **attr.docsExamined** serta nilai dari **attr.docsExamined** sendiri tidak terlalu besar.

- **attr.storage.data.bytesRead**: Jumlah total *bytes* yang di-*read* dari diska saat menjalankan suatu *query*. Tidak semua proses *query* memiliki atribut ini.

- **attr.durationMillis**: Durasi dari eksekusi suatu *query* dalam satuan *milliseconds*.

## Pengaturan Log Agent Promtail

Agar log dapat dikirim ke Loki dengan label yang ingin kita inginkan, maka terdapat beberapa pengaturan di *file* Promtail yang harus disesuaikan. Di bawah ini merupakan langkah untuk mengatur Promtail sebagai log *agent* di server target.

1. Buka *file* pengaturan Promtail.

    ```console
    sudo nano /etc/promtail/promtail-local-config.yaml
    ```

2. Lalu sesuaikan isinya dengan pengaturan di bawah ini.

    ```console
    server:
      http_listen_port: 9080
      grpc_listen_port: 0

    positions:
      filename: /tmp/positions.yaml

    clients:
      - url: http://<ip-private-loki>:3100/loki/api/v1/push

    scrape_configs:
    - job_name: <hostname>-mongodb
      static_configs:
      - targets:
          - localhost
        labels:
          job: <hostname>l-mongodb-logs
          dbHostname: <hostname>
          __path__: /var/log/mongodb/*log
      pipeline_stages:
          - json:
              expressions:
                timestamp: t.date
                message: msg
                namespace: attr.ns
                command: attr.command
                planSummary: attr.planSummary
                docsExamined: attr.docsExamined
                nReturned: attr.nreturned
                keysExamined: attr.keysExamined
                bytesRead: attr.storage.data.bytesRead
                duration: attr.durationMillis
          - timestamp:
              source: timestamp
              format: RFC3339
          - labels:
              timestamp:
              job:
              namespace:
              command:
              planSummary:
              docsExamined:
              keysExamined:
              bytesRead:
              duration:
          - output:
              source: message
    ```

3. Setelah diubah, silakan *restart* servis MongoDB.

    ```console
    sudo systemctl restart promtail.service
    ```

### Keterangan Singkat dari *File* Pengaturan Promtail.

- **http** -> **http_listen_port**: *Port* yang digunakan server untuk menjalankan Promtail.

- **clients** -> **url**: URL *endpoint* tempat Promtail akan mengirim log.

- **scrape_configs** -> **static_configs** -> **labels**: Menambahkan label khusus ke setiap log sebelum dikirim ke sistem *logging* terpusat. Label `__path__` merupakan label khusus yang menunjukkan lokasi sumber log berada.

#### Pipeline Stage

*Pipeline stage* merupakan bagian pengaturan yang berisi pemrosesan log sebelum dikirim ke Loki.Contoh pemrosesan log yang terjadi serta bisa diatur di bagian ini adalah *parsing*. Pengaturan ini berada di bawah pengaturan **scrape_configs**. Terdapat beberapa komponen pengaturan di bagian *pipeline stage*. Berikut penjelasan singkatnya.

- **json**: Mengekstrak nilai dari log yang berformat JSON. Nilai akan disimpan sebagai label.
- **timestamp**: Mengganti *timestamp* berdasarkan isi *timestamp* isi log bukan *timestamp* dari Promtail saat membaca log. Isi *timestamp* disesuaikan berdasarkan zona waktu server tempat servis MongoDB berada. RFC3339 adalah format standar timestamp yang digunakan dalam banyak sistem *logging*.
- **label**: Menentukan label mana yang akan dikirim ke Loki setelah nilainya berhasil diekstrak di pengaturan **json** di atas.
- **output**: Mengganti isi log setelah terjadinya pemrosesan log. Berdasarkan dari pengaturan di atas, isi log diganti menjadi "Slow query" sesuai dengan label **msg**. Berikut penampakan log *slow query* di Grafana yang sudah terkirim ke Loki berdasarkan pengaturan **output** dan **timestamp** di atas.
![log-slow-query-grafana](/assets/img/posts/devops/output-log-slow-query-di-grafana.png)
_Output Log Slow Query di Grafana_

## Visualisasi Log Slow Query di Grafana

Sebelum melakukan visualisasi log *slow query* pastikan Loki sudah terintegrasi ke Grafana. Dalam kasus ini, contoh visualisasi yang dibuat adalah mendapatkan 10 *queries* paling lambat berdasarkan durasinya. Berikut merupakan langkah-langkah membuat visualisasi log 10 *queries* paling lambat berdasarkan durasi di Grafana.

### Pengaturan Dashboard

1. Pada bagian navigasi menu di sebelah kiri aplikasi Grafana, pilih **Dashboards**.
2. Klik **New**, lalu pilih **New dashboard**.
3. (Opsional) Atur variabel pada *dashboard*. Klik **Settings**, lalu masuk ke tab **Variables**. Klik **Add Variable**. Isi/pilihlan nilai dari formulir inputnya sebagai berikut. lalu sisanya biarkan saja dengan pengaturan *default-nya*.
   - **Select variable type**: Query
   - **Name**: hostname
   - **Data source**: loki (disesuaikan dengan nama *data source* Loki yang diintegrasikan ke Grafana)
   - **Query type**: Label values; **Label**: dbHostname

   Biarkan pengaturan lainnya secara *default* lalu klik **Run query**.
    > Penambahan variabel di atas berguna jika kita memiliki lebih dari satu server target yang ingin kita atur lognya, dimana tiap servernya berisi servis MongoDB. Sesuai pengaturan Promtail di atas, terdapat penambahan label dbHostname yang berisi `<hostname>` server target. Dengan menggunakan variabel, *dashboard* tersebut dapat melakukan filter berdasarkan label dbHostname.

### Pengaturan Kueri dan Bentuk Visualisasi
1. Klik **Back to dashboard**, lalu klik **Add visualization**.
2. Pilih loki saat *wizard* **Select data source** muncul.
3. Pada bagian bawah pengaturan, terdapat 3 tab, yaitu **Queries**, **Transformations**, dan **Alert**. Pilih bagian **Queries**.
4. Pilih tab **Code**, lalu masukkan kueri berikut.
    ```console
    {dbHostname="$hostname"} |= `Slow query`
    ```
    Kueri di atas menampilkan log yang isi kontennya berisi kata "Slow query".
5. Pilih **Table** sebagai tipe visualisasi yang akan digunakan.
6. Pada **Title** di bagian **Panel options** di sebelah kanan aplikasi Grafana, isi dengan Top 10 Slowest Queries Log by Duration. Silahkan isinya disesuaikan dengan Anda.
7. Jalankan **Run query** di bagian tab **Queries** pada bawah pengaturan.
Berikut penampakan hasil dari pengaturan di atas.
![visualisasi-log-slow-query](/assets/img/posts/devops/visualisasi-pengaturan-kueri-log-slow-query.png)
_Visualisasi Pengaturan Kueri Log Slow Query di Grafana_

### Transformasi Data

1. Pilih tab **Transformations** pada bagian bawah pengaturan.
2. Klik **Add transformation**. Lalu *search* dan pilih **Extract fields**. Pada bagian **Source** pilih {} labels.
    > Perhatikan gambar di atas, pada kolom labels di tabel tersebut berisi semua label yang telah di-*parsing* oleh log *agent*. Tujuan tranformasi ini adalah untuk mengekstrak nilai pada kolom *labels* menjadi *field* atau kolom baru.
    
    *Enable* pengaturan **Replace all fields** dan **Keep time**.
3. Klik **Add another transformation**. Lalu *search* dan pilih **Convert field type**. Pada bagian **Field** pilih bagian duration dan **as** Number.
    > Secara *default*, semua kolom baru yang sudah melalui proses transformasi ekstraksi di atas bertipe String.
4. Klik **Add another transformation**. Lalu *search* dan pilih **Sort by**. Pada bagian **Field** pilih duration dan *enable* pengaturan **Reverse**.
    > Transformasi ini bertujuan untuk menampilkan data (baris) berdasarkan durasi paling tinggi.
5. Klik **Add another transformation**. Lalu *search* dan pilih **Organize fields by name**. *Disable* apa saja kolom yang tidak diperlukan di visualisasi tabel. Secara *default*, semua kolom hasil ekstraksi ditampilkan di visualisasi tabel. Penamaan kolom juga bisa diatur pada tahap transformasi ini.
6. Klik **Add another transformation**. Lalu *search* dan pilih **Limit**. Atur nilai batasan maksimal kueri yang ingin ditampilkan di visualisasi tabel.
7. Klik **Back to dashboard**, lalu atur posisi dan ukuran visualisasi *dashboard*. Terakhir klik **Save dashboard**. Berikut penampakan akhir dari visualisasi log *slow query*.
![visualisasi-log-slow-query](/assets/img/posts/devops/visualisasi-akhir-log-slow-query.jpg)
_Visualisasi Akhir Log Slow Query di Grafana_