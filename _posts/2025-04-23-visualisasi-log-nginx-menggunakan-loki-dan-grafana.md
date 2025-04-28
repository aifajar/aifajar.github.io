---
title: Visualisasi Log Nginx (Access & Error) menggunakan Loki dan Grafana
date: 2025-04-23 23:59:59 +0700
categories: [DevOps]
tags: [devops, devops tools, logging system, nginx, promtail]
image:
  path: /assets/img/posts/devops/alur-request-pada-nginx.png
  alt: Alur Request pada Nginx
---

Sebelum mengikuti instruksi di bawah pastikan *tool* Promtail sudah terinstal di server target yang terdapat servis Nginx-nya. Jika belum memiliki Promtail, silakan Anda cek tulisan saya sebelumnya [di sini](https://blog.aifajar.com/posts/pengaturan-instalasi-loki-dan-promtail/) untuk mengikuti panduan instalasinya.

## Pengaturan Custom Log Access Servis Nginx

Pada pengaturan *default* servis Nginx, konten log *access* tidak terdapat informasi terkait waktu. Informasi terkait waktu bisa sangat berguna untuk melakukan diagnosis suatu isu jika suatu sistem mengalami lemot. Berikut merupakan contoh penampakan log *access* *(fake)* pada servis Nginx secara *default*.

![log-access-nginx-default](/assets/img/posts/devops/contoh-log-access-nginx-default.png)
_Contoh Log Access Nginx Default_

Terdapat beberapa tambahan variabel terkait waktu yang cukup penting untuk dimasukkan ke dalam konten log yang akan di-*custom* kali ini. Beberapa variabel yang ditambah, antara lain: **$upstream_connect_time**, **$upstream_header_time**, **$upstream_response_time**, dan **$request_time**. Tutorial ini hanya berfokus pada pembuatan visualisasi dengan menggunakan variabel **$request_time** saja. Referensi terkait *custom* log *access* servis Nginx, dapat Anda lihat [di sini](https://docs.nginx.com/nginx/admin-guide/monitoring/logging/).

*File* pengaturan terkait *custom* log *access* di servis Nginx terletak di direktori `/etc/nginx/nginx.conf` dan di direktori `/etc/nginx/sites-enabled/<pengaturan-domain>`. Berikut langkah untuk mengubah pengaturan *custom* log di servis Nginx.

1. Buka *file* pengaturan servis Nginx.

    ```console
    sudo nano /etc/nginx/nginx.conf
    ```

2. Pada bagian **Logging Settings**, tambahkan pengaturan berikut.

    ```console
        ...
        ##
        # Logging Settings
        ##

        log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                                '"$request" $status $body_bytes_sent '
                                '"$http_referer" "$http_user_agent" '
                                'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';
        ...
    ```

    Ubah pengaturan **access_log** yang posisinya tepat berada di bawah pengaturan di atas dari **/var/log/nginx/access.log** menjadi **off**.

    > Pengaturan ini ditujukan agar tidak semua domain diaktifkan log *access*-nya. Pengaktifan log *access* pada domain tertentu akan dibahas di bagian berikutnya.

3. Masuk ke direktori `/etc/nginx/sites-enabled/`, lalu pilih domain mana yang perlu di atur log *access*-nya, contoh domain example.id. 

    Buka *file* `/etc/nginx/sites-enabled/example.id`.

    ```console
    sudo nano /etc/nginx/sites-enabled/example.id
    ```

    Pada bagian **server**, tambahkan dan sesuaikan berdasarkan contoh berikut.

    ```console
    ...
    server {
        ...
        access_log /var/log/nginx/example.id.access.log upstream_time;
        ...
    }
    ...
    ```

4. Verifikasi sintaksis pengaturan servis Nginx jika terdapat eror.

    ```console
    sudo nginx -t
    ```

5. Jika tidak terdapat eror pada langkah di atas, silakan *restart* servis Nginx dan cek statusnya.

    ```console
    sudo systemctl restart nginx.service
    sudo systemctl status nginx.service
    ```

    > [!WARNING]
    > Mengaktifkan log *access* pada domain servis Nginx yang memiliki *throughput* tinggi (jutaan *requests per second*) dapat menciptakan terlalu banyak log, sehingga mengakibatkan kapasitas penyimpanan diska di server target maupun *object storage* yang digunakan Loki menjadi besar. Log juga dapat meningkatkan penggunaan *write* I/O *disk* di server. Peningkatan penggunaan I/O *disk* tanpa diimbangi dengan spesifikasi diska dan *hardware resources* yang mumpuni dapat mengakibatkan penurunan performa kinerja server. Bisa dipertimbangkan agar hanya mengaktifkan log *access* sistem yang terdapat di lingkungan *development* dan *testing* saja. Hindari mengaktifkannya di lingkungan *production*!

## Struktur Log Access Nginx

Lokasi log *access* lengkap masing-masing domain berada di `/etc/nginx/sites-enabled/<domain>.access.log`. Berikut merupakan contoh penampakan struktur log yang sudah di-*custom* berdasarkan penerapan langkah-langkah di atas.

![log-access-nginx-custom](/assets/img/posts/devops/contoh-log-access-nginx-custom.png)
_Contoh Log Access Nginx Custom_

Berdasarkan contoh log (log pertama, paling atas) di atas, struktur variabel dan nilai dari log dapat diekstrak sebagai berikut.

```console
remote_addr             : 180.242.21.43
remote_user             : -
time_local              : 09/Apr/2025:15:15:31 +0700
method                  : GET
request                 : /api/v1/ClassSchool/statInfo?filter=%7B%22schoolId%22:%2121295e60a1b74c10b8f96949%22,%22year%22:%222024%22%7D
protocol                : HTTP/2.0
status                  : 200
body_bytes_sent         : 22
http_referer            : http_referer
http_user_agent         : Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36
request_time            : 0.096
upstream_connect_time   : 0.000
upstream_header_time    : 0.000
upstream_response_time  : 0.096
```

Beberapa variabel dan nilai pada log di atas akan di-*parsing* dan dikirim sebagai label di Loki.

## Analisis Singkat Log Access Nginx

Berikut merupakan penjelasan singkat dari beberapa label yang diperlukan dalam proses analisis log.

- **remote_addr**: IP *address user* atau klien yang melakukan *request* ke server.

- **method**: Jenis metode HTTP yang digunakan pada *request*. Contohnya adalah GET, POST, PUT, DELETE, dan OPTIONS.

- **request**: URL lengkap *endpoint* dari suatu *request*, termasuk juga *query string*-nya.

- **status**: Status kode HTTP yang dikembalikan oleh server.

- **body_bytes_sent**: Jumlah *byte* data respons pada *body* yang dikirim ke klien, tidak termasuk *header*.

- **http_referer**: URL asal *request* berasal.

- **http_user_agent**: *Browser* atau perangkat apa yang digunakan klien.

- **request_time**: Waktu total yang dibutuhkan untuk memproses sebuah *request*.

- **upstream_connect_time**: Waktu yang dibutuhkan untuk membuat koneksi ke server *upstream*.
    > Server *upstream* merupakan server yang menerima dan memproses *request* melalui servis Nginx. Contohnya, servis Nginx akan meneruskan *request* ke server *upstream* yang berupa servis *backend* yang berjalan di port 3000. Lalu servis *backend* akan mengirim respons ke klien melalui servis Nginx.

- **upstream_header_time**: Waktu yang dibutuhkan untuk menerima *byte* data pertama respons *header* dari server *upstream*.

- **upstream_response_time**: Waktu yang dibutuhkan untuk menerima seluruh *byte* data respons *body* dari server *upstream*.

Pembagian **request_time** dan **upstream_response_time** dapat digunakan untuk menentukan apakah respons yang lambat berasal dari servis Nginx atau servis *backend*. Berikut merupakan ilustrasi alur suatu *request* pada servis Nginx.

![alur-request-nginx](/assets/img/posts/devops/alur-request-pada-nginx.png)
_Alur Request pada Nginx_

Nilai **request_time** ditunjukkan oleh waktu yang dibutuhkan dari proses 1 sampai 4. Sedangkan **upstream_response_time** ditunjukkan pada proses 2 dan 3.

## Pengaturan Log Agent Promtail

Agar log dapat dikirim ke loki dengan label yang kita inginkan, maka terdapat beberapa pengaturan yang perlu di atur di *file* pengaturan Promtail. Di bawah ini merupakan langah untuk mengatur Promtail sebagai log *agent* di server target.

1. Buka *file* pengaturan Promtail.

    ```console
    sudo nano /etc/promtail/promtail-local-config.yaml
    ```

2. Lalu sesuaikan dengan pengaturan di bawah ini.

    ```console
    server:
    http_listen_port: 9080
    grpc_listen_port: 0

    positions:
    filename: /tmp/positions.yaml

    clients:
    - url: http://<ip-private-loki>:3100/loki/api/v1/push

    scrape_configs:
    - job_name: <hostname>-nginx-access
    static_configs:
    - targets:
        - localhost
        labels:
        job: <hostname>-nginx-access-logs
        webHostname: <hostname>
        __path__: /var/log/nginx/*ccess.log
    pipeline_stages:
        - regex:
            expression: '^(?P<remote_addr>\S+) - (?P<remote_user>\S+) \[(?P<time_local>[^\]]+)\] "(?P<method>\S+) (?P<request>[^ ]+) (?P<protocol>[^"]+)" (?P<status>\d{3}) (?P<body_bytes_sent>\S+) "(?P<http_referer>[^"]*)" "(?P<http_user_agent>[^"]*)" rt=(?P<request_time>[\d\.]+) uct="(?P<upstream_connect_time>[\d\.]+|-)" uht="(?P<upstream_header_time>[\d\.]+|-)" urt="(?P<upstream_response_time>[\d\.]+|-)"$'
        - labels:
            remote_addr:
            method:
            request:
            status:
            body_bytes_sent:
            http_referer:
            http_user_agent:
            request_time:
            upstream_connect_time:
            upstream_header_time:
            upstream_response_time:
    - job_name: <hostname>-nginx-error
    static_configs:
    - targets:
        - localhost
        labels:
        job: <hostname>-nginx-error-logs
        webHostname: <hostname>
        __path__: /var/log/nginx/error.log
    ```

3. Setelah diubah, silakan *restart* Promtail.

    ```console
    sudo systemctl restart promtail.service
    ```

### Keterangan Singkat dari *File* Pengaturan Promtail.

- **http**->**http_listen_port**: *Port* yang digunakan server untuk menjalankan Promtail.

- **clients**->**url**: URL *endpoint* tempat Promtail akan mengirim log ke Loki.

- **scrape_configs**->**static_configs**->**labels**: Menambahkan label khusus ke setiap log sebelum dikirim ke Loki. Label `__path__ ` merupakan label khusus yang menunjukkan lokasi sumber log berada. Dalam kasus ini, saya menggunakan kedua jenis log yang terdapat di servis Nginx, yaitu semua log *access* dan sebuah log *error*.

#### Pipeline Stage

*Pipeline stage* merupakan bagian pengaturan yang berisi pemrosesan log sebelum dikirim ke Loki. Contoh pemrosesan log yang terjadi serta bisa diatur di bagian ini adalah parsing. Pengaturan ini berada di bawah pengaturan **scrape_configs**. Terdapat beberapa komponen pengaturan di bagian *pipeline stage*. Berikut penjelasan singkatnya.

- **regex**: Mengekstrak nilai dari log menggunakan *regular expression*. Nilai yang diekstrak akan disimpan sebagai label di Loki. Atribut **expression** merupakan pola dari *regular expression* untuk dicocokkan dengan konten log. Ekspresi umum `(?P<name>pattern)` menunjukkan bagian dari konten log yang akan disimpan pada variabel name. Bagian `pattern` merupakan pola karakter yang akan dicocokkan pada konten log lalu disimpan di variabel yang telah ditentukan. Berikut beberapa penjelasan singkat contoh ekspresi dari bagian *regular expression* yang terdapat di pengaturan di atas.

    - Ekspresi `[^\]]+` pada bagian `(?P<time_local>[^\]]+)` akan mencocokkan karakter apapun asalkan bukan tanda `]`. Tanda `\[` dan `\]` pada sebelum dan sesudah bagian tersebut bertujuan untuk mencocokkan konten log yang memiliki tanda-tanda tersebut.

    - Ekspresi `\S+` pada bagian `(?P<method>\S+)` akan mencocokkan karakter non-spasi.

    - Ekspresi `[^ ]+` pada bagian `(?P<request>[^ ]+)` akan mencocokkan karakter apapun selain spasi.

    - Ekspresi `[\d\.]+` pada bagian `(?P<request_time>[\d\.]+)` akan mencocokkan karakter angka (0-9) atau titik (.).

- **labels**: Menentukan label mana yang akan dikirim ke Loki setelah nilainya berhasil diekstrak menggunakan *regular expression* di atas.

> Dalam kasus ini, log *error* pada servis Nginx tidak perlu dilakukan *parsing* karena konten lognya tidak berpola dan saya hanya ingin menampilkan konten log *error* Nginx secara utuh di Grafana.

## Visualisasi Log Nginx di Grafana

Sebelum melakukan visualisasi log pada servis Nginx, pastikan Loki sudah terintegrasi ke Grafana. Berikut beberapa visualisasi yang dibuat dari informasi log Nginx yang ada.

### Log Error Nginx

Visualisasi yang ditampilkan untuk log *error* Nginx berisi keseluruhan konten log tanpa di-*parsing*. Berikut merupakan langkah-langkah pembuatan visualisasi ini.

1. Pada bagian navigasi menu di sebelah kiri aplikasi Grafana, pilih **Dashboards**.

2. Klik **New**, lalu pilih **New dashboard**.

3. Klik **Add visualization**.

4. Pilih loki saat *wizard* **Select data source** muncul.

5. Pada bagian bawah pengaturan, terdapat 3 tab, yaitu **Queries**, **Transformations**, dan **Alert**. Pilih bagian **Queries**.

6. Pilih tab **Code**, lalu masukkan kueri berikut.

    ```console
    {filename="/var/log/nginx/error.log", webHostname="<hostname>"} |= ``
    ```

7. Pilih **Logs** sebagai tipe visualisasi yang akan digunakan.

8. Pada **Title** di bagian **Panel options** di sebelah kanan aplikasi Grafana. Dalam kasus ini, isi dengan Nginx Log Error. Silahkan isinya disesuaikan dengan Anda.

9. Jalankan **Run query** di bagian tab **Queries**.

> Walaupun tanpa di-*parsing* di Promtail, terdapat label **level** yang terbentuk secara otomatis, yang menunjukkan level dari sebuah log. Level pada log *error* di servis Nginx memiliki level yang sama dengan log-log lain pada umumnya, seperti *warn*, *error*, *crit*, *alert*, sampai *emerg*. Berikut bentuk penampakan dari visualisasi dari log *error* berikut.

![visualisasi-log-error-nginx](/assets/img/posts/devops/visualisasi-log-error-nginx.png)
_Visualisasi Log Error Nginx_

### Log Access Nginx

Terdapat beberapa informasi yang dapat ditampilkan dari visualisasi log *access* di Nginx. Berikut beberapa contoh diantaranya.

#### Informasi Requests Paling Lambat berdasarkan Durasi

Berdasarkan pengaturan Nginx di atas, setiap domain akan membuat masing-masing log *access*-nya. Akan dibuat filter berdasarkan file log *access* yang ada di server. *File* log *access* ditunjukkan oleh label **filename** yang otomatis terbuat di Loki. Format dari isian label tersebut berupa `/var/log/nginx/<domain>.access.log`. Berikut langkah-langkah pembuatan visualisasinya.

##### Pengaturan Dashboard

1. Pada bagian navigasi menu di sebelah kiri aplikasi Grafana, pilih **Dashboards**.

2. Klik **New**, lalu pilih **New dashboard**.

3. Atur variabel pada *dashboard*. Klik **Settings**, lalu masuk ke tab **Variables**. Klik **Add Variable**. Isi/pilihlan nilai dari formulir inputnya sebagai berikut. lalu sisanya biarkan saja dengan pengaturan *default*-nya.
    - **Select variable type**: Query
    - **Name**: filename
    - **Data source**: loki (disesuaikan dengan nama data source Loki yang diintegrasikan ke Grafana)
    - **Query type**: Label values; Label: dbHostname

##### Pengaturan Kueri dan Bentuk Visualisasi

1. Klik **Back to dashboard**, lalu klik **Add visualization**.

2. Pilih loki saat *wizard* **Select data source** muncul.

3. Pada bagian bawah pengaturan, terdapat 3 tab, yaitu **Queries**, **Transformations**, dan **Alert**. Pilih bagian **Queries**.

4. Pilih tab **Code**, lalu masukkan kueri berikut.

    ```console
    {filename="$filename"} |= ``
    ```

5. Pilih **Table** sebagai tipe visualisasi yang akan digunakan.

6. Pada **Title** di bagian **Panel options** di sebelah kanan aplikasi Grafana, isi dengan Top Slowest Requests by Time. Silahkan isinya disesuaikan dengan Anda.

7. Jalankan **Run query** di bagian tab **Queries**.

##### Transformasi Data

1. Pilih tab **Transformations** pada bagian bawah pengaturan.

2. Klik **Add transformation**. Lalu *search* dan pilih **Extract fields**. Pada bagian **Source** pilih {} labels.

    > Tujuan tranformasi ini adalah untuk mengekstrak nilai pada kolom labels menjadi field atau kolom baru.

    *Enable* pengaturan **Replace all fields** dan **Keep time**.

3. Klik **Add another transformation**. Lalu *search* dan pilih **Convert field type**. Pada bagian **Field** pilih bagian request_time dan as Number.

    > Secara *default*, semua kolom baru yang sudah melalui proses transformasi ekstraksi di atas (nomor 2) bertipe String.

4. Klik **Add another transformation**. Lalu *search* dan pilih **Sort by**. Pada bagian **Field** pilih request_time dan *enable* pengaturan **Reverse**.

    > Transformasi ini bertujuan untuk menampilkan data (baris) berdasarkan durasi paling tinggi.

5. Klik **Add another transformation**. Lalu *search* dan pilih **Organize fields by name**. *Disable* apa saja kolom yang tidak diperlukan di visualisasi tabel. Secara *default*, semua kolom hasil ekstraksi ditampilkan di visualisasi tabel. Penamaan kolom juga bisa diatur pada tahap transformasi ini.

6. Klik **Add another transformation**. Lalu *search* dan pilih **Limit**. Atur nilai batasan maksimal kueri yang ingin ditampilkan di visualisasi tabel.

7. Klik **Back to dashboard**, lalu atur posisi dan ukuran visualisasi *dashboard*. Terakhir klik **Save dashboard**.

#### Informasi Requests Endpoint yang Paling Banyak Diakses

Berawal dari migrasi *tool* terkait *observability* bagian *logging*, dari New Relic ke Loki, saya memiliki visualisasi terkait informasi *request* yang paling banyak diakses atau di-*hit* oleh *client* melalui sumber log *access* di servis Nginx. Sayapun ingin membuat visualisasi tiruan yang ekuivalen dengan kueri di New Relic sebagai berikut.

```console
SELECT count(*), average(request_time) FROM Log FACET request, response, verb WHERE hostname = '<hostname>' AND filePath = '/var/log/nginx/<domain>.access.log SINCE 1 hour AGO'
```

Label `response` pada New Relic merupakan status *code* pada label di *project* ini dan `verb` merupakan *method*. `FACET` pada NRQL (New Relic Query Language) ekuivalen dengan GROUP BY pada SQL. 

>Jujur saja, saya kurang begitu yakin apakah implementasi visualisasi berikut sudah akurat dengan tujuan yang saya maksud. Namun, visualisasi ini bisa dibilang mendekati dengan yang saya buat di New Relic sesuai kueri di atas.

Perbedaan implementasi antara visualisasi ini dan sebelumnya terletak pada proses transformasi data. Untuk membuat visualisasi ini, silakan ulangi proses Pengaturan Kueri dan Bentuk Visualisasi pada bagian atas, lalu sesuaikan bagian **Title**-nya. Dalam kasus ini, saya menggunakan Top Hit Endpoint Requests, silakan sesuaikan dengan keinginan Anda. Selanjutnya, silakan ikuti proses berikut.

##### Transformasi Data

1. Pilih tab **Transformations** pada bagian bawah pengaturan.

2. Klik **Add transformation**. Lalu *search* dan pilih **Extract fields**. Pada bagian **Source** pilih {} labels.

    *Enable* pengaturan **Replace all fields** dan **Keep time**.

3. Klik **Add another transformation**. Lalu *search* dan pilih **Organize fields by name**. *Disable* apa saja kolom yang tidak diperlukan di visualisasi tabel. Dalam kasus ini, kolom yang diperlukan antara lain:
    - **request** di-*rename* menjadi Request.
    - **method** di-*rename* menjadi Method.
    - **status** di-*rename* menjadi Status Code.
    - **request_time** di-*rename* menjadi Request Time.

4. Klik **Add another transformation**. Lalu *search* dan pilih **Convert field type**. Pada bagian Field pilih bagian request_time (base field name) dan as Number.

5. Klik **Add another transformation**. Lalu search dan pilih **Add field from calculation**. Isi pengaturan dengan pilihan atau isian berikut.
    - **Mode**: Reduce row
    - **Operation**: Request
    - **Calculation**: Distict Count
    - **Alias**: Total Hits
    > Pengaturan **Replace all fields** jangan di-*enable*.

    > Transformasi pada bagian ini bertujuan untuk membuat kolom baru yang berisi total *hit endpoint request*.

6. Klik **Add another transformation**. Lalu search dan pilih **Group by**. Isi pengaturan dengan pilihan berikut.
    - **Request**: Group by
    - **Method**: Group by
    - **Status Code**: Group by
    - **Request Time**: Calculate, lalu pilih Mean.
    - **Total Hits**: Calculate, lalu pilih Total.
    > Transformasi pada bagian ini bertujuan untuk melakukan operasi agregasi untuk mencari rata-rata *request time* dan total *hit* berdasarkan *request*, *method*, dan *status code*.

7. Klik **Add another transformation**. Lalu *search* dan pilih **Sort by**. Pada bagian **Field** pilih Total Hits (sum) dan *enable* pengaturan **Reverse**.

8. Klik **Back to dashboard**, lalu atur posisi dan ukuran visualisasi *dashboard*. Terakhir klik **Save dashboard**.

Penampakan visualisasi untuk informasi *requests* endpoint yang paling banyak diakses dapat dilihat pada gambar di bawah ini.

![visualisasi-top-hit-endpoint-requests](/assets/img/posts/devops/visualisasi-top-hit-endpoint-requests.png)
_Visualisasi Top Hit Endpoint Requests_






