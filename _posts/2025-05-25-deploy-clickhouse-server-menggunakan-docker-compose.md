---
title: Deploy Clickhouse-Server menggunakan Docker Compose
date: 2025-05-25 23:59:59 +0700
categories: [DevOps]
tags: [devops, data engineering, clickhouse, docker]
---

Clickhouse merupakan sistem *database* yang *open source* dan bertipe *column-oriented*. Dirancang untuk memproses data analitik dalam jumlah besar dengan kecepatan tinggi, ClickHouse banyak digunakan untuk analitik *real-time analytics* dan pelaporan. *Column-oriented database* memungkinkan eksekusi kueri lebih cepat pada *dataset* yang besar sehingga sangat cocok untuk penggunaan sistem OLAP (*Online Analytical Processing*).

Beberapa keunggulan lain dari ClickHouse selain performa kecepatan kueri adalah *developer friendly* dan biaya yang efisien. Dikarenakan keunggulannya, ClickHouse sendiri sering dijadikan sebagai alternatif *tools* OLAP lain, seperti Google BigQuery, Snowflake, Databricks, dan Amazon RedShift.

## Langkah Deploy ClickHouse

Pada tutorial ini, saya menggunakan Ubuntu sebagai sistem operasi untuk menjalankan server ClickHouse. Cara instalasi yang dipakai adalah menggunakan Docker Compose. Docker Compose digunakan karena alasan kemudahan, kecepatan, pemeliharaan, serta fleksibilitas yang ditawarkan dibanding dengan cara instalasi manual. Proses *upgrade* dan penambahan *tools* lain (seperti *tool monitoring*) juga hanya perlu menambahakan beberapa baris saja. Berikut merupakan langkah-langkah instalasi ClickHouse menggunakan ClickHouse menggunakan Docker Compose.

### Instalasi ClickHouse

1. Pastikan Docker Engine terinstal di server Anda. Dalam kasus ini, saya menggunakan sistem operasi Ubuntu, jadi bisa merujuk ke tautan berikut: https://docs.docker.com/engine/install/.

2. Masuk ke direktori `/opt`, lalu buat direktori baru dan *file* Docker Compose bernama docker-compose.yml.

    ```
    cd /opt
    mkdir -p clickhouse && cd clickhouse
    nano docker-compose.yml
    ```

    Lalu isi file tersebut dengan pengaturan di bawah ini.

    ```console
    version: '3'
    services:
      clickhouse-server:
        image: clickhouse/clickhouse-server:25.3
        restart: always
        container_name: clickhouse-server
        hostname: clickhouse-server
        environment:
          - TZ=Asia/Jakarta
          - CLICKHOUSE_USER=admin
          - CLICKHOUSE_PASSWORD=<isi-sendiri>
          - CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1
        ulimits:
          nofile:
            soft: 262144
            hard: 262144
        ports:
          - '8123:8123'
          - '9000:9000'
        volumes:
          - /clickhouse/data:/var/lib/clickhouse
          - /clickhouse/logs:/var/log/clickhouse-server
        network_mode: host
        cap_add:
        - SYS_NICE
        - NET_ADMIN
        - IPC_LOCK
        - SYS_PTRACE
    ```

    Keterangan:
    
    - `image: clickhouse/clickhouse-server:25.3`: Nama dan versi *container image* yang digunakan. Versi 25.3 adalah versi LTS terbaru ketika tulisan ini dibuat.

    - `restart: always`: *Container* akan selalu otomatis restart jika berhenti secara tak terduga.

    - `container_name: clickhouse-server`: Nama proses *container* yang sedang berjalan di Docker.

    - `environments`: Variabel lingkungan untuk *container*.
      1. `TZ=Asia/Jakarta`: Mengatur *timezone container* ke Asia/Jakarta.
      2. `CLICKHOUSE_USER=admin`: Membuat *user* bertipe admin bernama `admin`.
      3. `CLICKHOUSE_PASSWORD=<isi-sendiri>`: *Password* untuk *user* admin.
      4. `CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1`: Mengaktifkan manajemen *user/role* di ClickHouse (sejak versi 20.4+).

    - `ulimits: ...`: Mengatur batasan *file descriptor* untuk *container*. Nilai *default* pada Ubuntu adalah 1024 (soft) dan 4096 (hard). ClickHouse memerlukan `nofile` yang tinggi untuk menangani banyak koneksi agar tidak gagal saat banyak koneksi masuk. Pengaturan di atas akan meningkatkan jumlah *file/connection* yang bisa dibuka.

    - `volumes: ...`: *Mount* direktori untuk menyimpan data dan log agar tetap *persistent* ketika *conatiner* dimatikan atau dihapus.

    - `network_mode: host`: *Container* akan menggunakan *network host* langsung (tidak terisolasi) di mana IP *container* sama dengan IP *host* tempat Docker berada.

    - `cap_add: ...`: Mengaktifkan kapabilitas Linux tambahan untuk *container* ClickHouse.
      1. `SYS_NICE` → Mengatur prioritas proses (`nice level`).
      2. `NET_ADMIN` → Akses pengaturan jaringan.
      3. `IPC_LOCK` → Mengunci memori (untuk performa).
      4. `SYS_PTRACE` → Digunakan untuk *debugging/profiling*.

3. Buat direktori `/clickhouse` pada server untuk menyimpan data dan log ClickHouse agar *persistent*.

```console
mkdir -p /clickhouse
mkdir -p /clickhouse/data /clickhouse/logs
```

4. Jalankan *command* berikut untuk menjalankan seluruh proses pengaturan *file* di atas.

    ```console
    docker compose up -d
    ```

5. Cek status proses server ClickHouse di Docker.

    ```console
    docker ps
    ```

### Verifikasi Hasil Deploy ClickHouse

Setelah *deploy* berhasil, pengecekan instalasi server ClickHouse dapat diverifikasi sebagai berikut.

- Jalankan *command* berikut untuk masuk ke sistem server ClickHouse.
    ```console
    docker exec -it clickhouse-server clickhouse-client
    ```
- Tampilkan daftar *database*
    ```console
    SHOW DATABASES
    ```

    Alternatifnya, Anda dapat menjalankan *command* di atas melalui browser melalui URL: `http://<ip-public-server>:8123/play`. Lalu masukkan informasi akun *username* dan *password* inisialisasi yang terdapat di *file* **docker-compose.yml** sebelumnya. Tampilannya kurang lebih seperti gambar di bawah.

    ![tampilan-clickhouse-di-browser](/assets/img/posts/devops/jalankan-command-clickhouse-di-browser.png)
    _Tampilan ClickHouse di Browser Setelah Terinstal_

    > Untuk menjalankan versi alternatif di *browser*, pastikan port 8123 dapat diakses melalui publik. Membuka port 8123 ke publik **sangat berisiko** terhadap keamanan data Anda. Silakan gunakan alternatif seperti menambahkan domain melalui *reverse proxy* agar bisa mengaktifkan HTTPS.

### *File* Pengaturan ClickHouse

Terdapat dua tipe *file* pengaturan utama di server Clickhouse, yaitu **config** dan **users**. Sesuai namanya, *file* tersebut digunakan untuk mengatur bagaimana pengaturan konfigurasi dan pengguna server ClickHouse berjalan.

Anda dapat menyalin *file* pengaturan dari dalam *container* yang sedang berjalan ke *host* tempat Docker terinstal, dalam hal ini ke direktori yang sama tempat *file* `docker-compose.yml` berada, yakni di `/opt/clickhouse`. Sebagai contoh, kita ingin memindahkan *file* bernama **users.xml** yang biasanya berada di `/etc/clickhouse-server/users.xml` dari dalam *container* ClickHouse yang berjalan ke direktor `/opt/clickhouse`. 

Hal ini dimaksudkan agar ketika kita ingin melakukan perubahan pengaturan terkait pengguna, kita tinggal mengubahnya di `/opt/users.xml`, lalu matikan serta hapus *container* lama dan jalankan ulang *container* baru dengan pengaturan terbaru. Kita akan membuat *file* `/opt/clickhouse/users.xml` *persistent* sehingga kita perlu mendefinisikannya di bagian volume pada *file* `docker-compose.yml`.

Pertama-tama, jalankan *command* berikut untuk mendapatkan *full path* dari *container* ClickHouse yang sedang berjalan.

```console
docker inspect clickhouse-server | grep -i merged
```

Salin *output* pada *command* tersebut mulai dari `var/lib/.../merged`. Lalu *copy file* ke direktori `/opt/clickhouse`.

```console
cp /var/lib/docker/overlay2/<your_copy_path>/merged/etc/clickhouse-server/users.xml /opt/clickhouse/
```

*Update file* `docker-compose.yml`, tambahkan *file* `/opt/clickhouse/users.xml` ke bagian volume agar *persistent*.

```console
version: '3'
services:
  clickhouse-server:
    image: clickhouse/clickhouse-server:25.3
    ...
    volumes:
      - /clickhouse/data:/var/lib/clickhouse
      - /clickhouse/logs:/var/log/clickhouse-server
      - 
    ...
```

Matikan dan hapus *container* yang sedang berjalan dengan *command* berikut:

```console
docker compose down
```

Lalu jalankan ulangkan dengan *command* berikut untuk membuat *container* dengan konfigurasi yang baru.

```console
docker compose up -d
```

> Cara di atas tidak direkomendasikan karena akan lebih baik untuk menambah pengaturan baru di direktori `/etc/clickhouse/config.d/` dan `/etc/clickhouse/users.d/` tanpa mengubah apapun pengaturan *default* dari *file*-nya. 

Berikut dokumentasi resmi terkait pengaturan kedua *file* tersebut.
- **Users Settings:** https://clickhouse.com/docs/operations/settings/settings-users
- **Configuration Files:** https://clickhouse.com/docs/operations/configuration-files

## Pengaturan ClickHouse Setelah Deploy

Berikut beberapa contoh pengaturan ClickHouse setelah berhasil di-*deploy* di server Anda.

### Buka Port Tertentu

ClickHouse kompatibel dengan protokol PostgreSQL dan MySQL sehingga dapat dibuka melalui aplikasi *client* mereka. Agar tersambung dengan aplikasi PostgreSQL maupun MySQL *clent*, silahkan *publish port* 9004 dan 9005. Melalui *port* 9004, ClickHouse akan dianggap sebagai MySQL apapun aplikasi yang tersambung melalui *port* ini. Begitupun dengan *port* 9005, ClickHouse akan dianggap sebagai PostgreSQL. Secara *default*, kedua *port* tersebut sudah digunakan oleh ClickHouse. Agar bisa diakses secara *remote* melalui aplikasi PostgreSQL/MySQL *client*, silahkan tambahkan daftar *port* yang ingin di-*publish* di bagian `ports` pada *file* `docker-compose.yml`.

```console
version: '3'
services:
  clickhouse-server:
    image: clickhouse/clickhouse-server:25.3
    ...
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
    ports:
      - '8123:8123'
      - '9000:9000'
      - '9004:9004'
      - '9005:9005'
    volumes:
      ...
```

Matikan dan hapus *container* yang sedang berjalan dengan *command* berikut:

```console
docker compose down
```

Lalu jalankan ulangkan dengan *command* berikut untuk membuat *container* dengan konfigurasi yang baru.

```console
docker compose up -d
```

Anda dapat tersambung ke ClickHouse melalui aplikasi MySQL *client* dengan menjalankan *command* berikut.

```console
mysql -h <ip-server-clickhouse> -P 9004 -u admin -p
```
Aplikasi *client* akan meminta *password* akun ClickHouse. Silahkan gunakan *password* yang Anda definisikan di *file* `docker-compose.yml`.

> Pastikan server ClickHouse sudah menambah IP dari aplikasi *client* berada ke dalam *whitelists* agar bisa tersambung. Jika sudah tersambung, jalankan *command* `nc -zv <ip-server-clickhouse> 9004`. Jika pesan yang muncul berisi `succeeded!`, maka koneksi antara aplikasi MySQL *client* ke ClickHouse sudah dapat tersambung. Untuk kasus ini, aplikasi MySQL *client* dijalankan dengan sistem operasi Ubuntu.

### Membuat Database dan Akun

Untuk mengatur proses terkait administrasi terkait ClickHouse, Anda dapat menggunakan *command* SQL. Sebagai contoh, jika Anda ingin membuat *database* baru bernama **ecommerce** dan **school**, maka *command* yang digunakan adalah sebagai berikut.

```console
CREATE DATABASE ecommerce;
```

Anda ingin membuat akun *user* khusus bernama **admin_ecommerce** agar bisa mengakses *database* **ecommerce** dan melakukan beberapa aksi. Aksi yang diperlukan antara lain:

- CRUD pada baris di *database* **ecommerce**.
- Mengubah & menghapus struktur tabel di *database* **ecommerce**.

Maka deretan *command* yang perlu dijalankan adalah sebagai berikut.

```console
CREATE USER admin_ecommerce IDENTIFIED WITH sha256_password BY '<your_secure_password>';
GRANT INSERT, SELECT, UPDATE, DELETE ON ecommerce.* TO admin_ecommerce;
GRANT ALTER ON ecommerce.* TO admin_ecommerce;
GRANT DROP TABLE ON ecommerce.* TO admin_ecommerce;
```

### Mengatur Pengaturan Session

Berdasarkan dokumentasi resminya, kita dapat mengatur pengaturan terkait *session* di server ClickHouse. Anda dapat meliha dokumentasinya [di sini](https://clickhouse.com/docs/operations/settings/settings). Pada tutorial ini, saya ingin mengatur *session* terkait HTTP *send* dan *received timeout*.

Pertama, pastikan Anda sudah masuk ke *container* server ClickHouse yang sedang jalan dengan menggunakan *clickhouse-client*. Cek nilai pengaturan `http_send_timeout` dengan *command* di bawah.

```console
SELECT value FROM system.settings where name='http_send_timeout';
```

Anda akan melihat nilainya adalah 30. Nilai tersebut merupakan nilai *default* pengaturannya yang sesuai dengan dokumentasi. Jika Anda ingin melihat `http_receive_timeout`, silahkan jalankan *command* berikut.

```console
SELECT value FROM system.settings where name='http_receive_timeout';
```

Nilainya juga sama dengan `http_send_timeout`. 

Pada tahap selanjutnya, kita akan melakukan *disable* terhadap kedua pengaturan *timeout* ini, sehingga server ClickHouse tidak memiliki *timeout* pada proses HTTP *send* dan *receive*. Untuk melakukan *disable*, silahkan ubah nilai `http_send_timeout` menjadi 0. Jalankan *command* berikut.

```console
SET http_send_timeout = 0;
```
> Untuk mengubah nilai pengaturan di server ClickHouse, pastikan baca dokumentasi resmi mereka yang sudah terdapat di bagian atas tulisan ini.

Cek nilai `http_send_timeout` kembali. Pastikan nilainya sudah berubah menjadi 0.

```console
SELECT value FROM system.settings where name='http_send_timeout';
```

Lakukan cara yang sama pada nilai `http_receive_timeout`.

```console
SET http_receive_timeout = 0;
```
Cek nilai `http_receive_timeout` kembali. Pastikan nilainya sudah berubah menjadi 0.

```console
SELECT value FROM system.settings where name='http_receive_timeout';
```