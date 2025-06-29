---
title: Pengaturan Instalasi Prometheus dan Grafana
date: 2025-06-30 23:59:59 +0700
categories: [DevOps]
tags: [devops, devops tools, monitoring system, prometheus, grafana]
---

Prometheus dan Grafana adalah *tools* populer terkait sistem monitoring. Beberapa alasan kepopulerannya antara lain adalah karena kedua *tools* tersebut merupakan *open source* dan mudah untuk digunakan. Prometheus digunakan untuk mengambil (*scrapping*) *metrics* yang ada pada suatu *endpoint* berdasarkan interval tertentu. *Endpoint* bisa berupa server maupun aplikasi. Sedangakan Grafana digunakan sebagai *tool* untuk melakukan visualisasi yang tidak hanya terbatas pada *metrics* yang dikumpulkan oleh Prometheus.

## Arsitektur Sistem Monitoring

Implementasi projects ini akan dibangun menggunakan arsitektur dari infrastuktur berikut.

![project-sistem-monitoring](/assets/img/posts/devops/arsitektur-project-sistem-monitoring.png)
_Arsitektur Project Sistem Monitoring_

*Projects* ini merupakan VM-based, jadi tidak menggunakan Docker atau *container engine* lain dalam penerapannya. Setiap satu ikon merepresentasikan sebuah server atau node yang berupa VM (*Virtual Machine*).

> Walaupun *project*-nya VM-*based*, tidak akan ada perbedaan yang signifikan bila coba diterapkan dengan menggunakan Docker.

## Informasi Lingkungan Project

Semua sistem operasi yang terdapat pada VM yang ada menggunakan Ubuntu LTS versi 24.04. Berikut beberapa informasi terkait lingkungan perangkat lunak *projects* yang digunakan.

| Sistem | Nama Aplikasi/*Tools*/dsb | Deskripsi | Versi |
|---|---|---|---|
| Server *Endpoint* | Node Exporter | *Agent* yang digunakan untuk mengumpulkan dan mengekspos *metrics* ke servis Prometheus. | 1.9.1 |
| Server Monitoring | Prometheus | Servis yang digunakan untuk mengambil *metrics* dari *agent*/*exporter*. | 2.53.4 |
| Server Monitoring | Grafana | Servis untuk visualisasi *metrics*. | 12.0.2 |
| *Database* | PostgreSQL/MySQL | *Servis* untuk menyimpan *database* dari Grafana. | PostgreSQL (16.9), MySQL (8.0.42) |

## Instalasi Prometheus

Berikut merupakan langkah instalasi dan pengaturan servis Prometheus.

1. Membuat *dedicated user* untuk servis Prometheus.

    ```console
    sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus
    ```

2. *Download package* Prometheus, contoh dalam dokumentasi ini menggunakan versi 2.52.0.

    ```console
    wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
    ```

    Ekstrak *file* Prometheus.

    ```console
    tar -xvf prometheus-2.53.4.linux-amd64.tar.gz
    ```
    Masuk ke direktori *file* hasil ekstrak.

    ```console
    cd prometheus-2.52.0.linux-amd64
    ```

3. Membuat direktori untuk servis Prometheus.

    ```console
    sudo mkdir -p /data /etc/prometheus
    ```

4. Pindahkan *files* Prometheus hasil ekstrak ke direktori-direktori terkait.

    ```console
    sudo mv prometheus promtool /usr/local/bin/
    sudo mv consoles/ console_libraries/ /etc/prometheus/
    sudo mv prometheus.yml /etc/prometheus/prometheus.yml
    ```

5. Atur *ownership* direktori-direktori terkait Prometheus.

    ```console
    sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
    ```

6. Verifikasi instalasi Prometheus.

    ```console
    prometheus --version
    ```

7. Atur Prometheus sebagai *systemd service*.

    ```console
    sudo nano /etc/systemd/system/prometheus.service
    ```

    Isi konten *file* yang barusan dibuat dengan pengauran berikut.

    ```console
    [Unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/data \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

    [Install]
    WantedBy=multi-user.target
    ```

8. *Start* dan *enable* servis Prometheus.

    ```console
    sudo systemctl start prometheus
    sudo systemctl enable prometheus
    ```

    Lalu cek servisnya.

    ```console    
    sudo systemctl status prometheus
    ```

### Instalasi Node Exporter

Node Exporter akan digunakan sebagai *agent* untuk mengumpulkan *metrics* terkait *host*/server berbasis Linux. Jika *host* menggunakan Windows Server, maka *agent*-nya dapat digantikan dengan menggunakan [Windows exporter](https://github.com/prometheus-community/windows_exporter). 

Beberapa *metrics* yang dapat dikumpulkan oleh Node Exporter, antara lain:
- Penggunaan CPU,
- penggunaan memori (RAM),
- kapasitas *disk*,
- besar *disk* I/O (*Input/Output*),
- besar *network transmit/receive*, 
- dsb.

Berikut langkah-langkah yang dilakukan untuk melakukan instalasi dan pengaturan Node Exporter di server *endpoint*.

1. Membuat sistem user untuk servis Node Exporter.

    ```console
    sudo useradd \
        --system \
        --no-create-home \
        --shell /bin/false node_exporter
    ```

2. *Download package* Node Exporter, contoh dalam dokumentasi ini menggunakan versi 1.9.1.

    ```console
    wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
    ```

    Ekstrak *file* Node Exporter.

    ```console
    sudo tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz
    ```

3. Pindahkan *file* binari Node Exporter ke direktori /usr/local/bin.

    ```console
    sudo mv node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/
    ```

4. Atur Node Exporter sebagai systemd service.

    ```console
    sudo nano /etc/systemd/system/node_exporter.service
    ```

    Isi konten *file* yang barusan dibuat dengan pengaturan berikut.

    ```console
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target
    StartLimitIntervalSec=500
    StartLimitBurst=5
    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/node_exporter \
        --collector.logind
    [Install]
    WantedBy=multi-user.target
    ```

5. *Start* dan *enable* servis Node Exporter.

    ```console
    sudo systemctl start node_exporter
    sudo systemctl enable node_exporter
    ```

    Lalu cek servisnya.

    ```console    
    sudo systemctl status node_exporter
    ```

6. Atur IP *whitelist* agar hanya Node Exporter bisa diakses oleh servis Prometheus saja.

    ```console
    sudo ufw allow from <ip-prometheus> to any port 9100
    ```

### File Pengaturan Prometheus

Berdasarkan cara instalasi dan pengaturan di atas, *file* pengaturan servis Prometheus terletak di /etc/prometheus/prometheus.yml. Kita bisa menggunakan berbagai macam *exporter* yang tersedia untuk berbagai aplikasi/*endpoint*, sesuai dengan kebutuhan, serta cara instalasi yang hampir mirip seperti Node Exporter di atas.

Setelah kita berhasil menginstalasi *exporter* di *endpoint* yang digunakan. Hal selanjutnya adalah memasukkan informasi terkait *endpoint* ke *file* pengaturan servis Prometheus. Berikut merupakan langkah selanjutnya untuk mengatur integrasi antara Node Exporter dan Servis Prometheus.

1. Silakan buka *file* pengaturan servis Prometheus.

    ```console
    sudo nano /etc/prometheus/prometheus.yml
    ```

2. Tambahkan Node Exporter di atas sebagai target di servis Prometheus, letakkan di paling bawah *file* pengaturan.

    ```console
    ...
    - job_name: '<server-name>-node_exporter'
        static_configs:
        - targets: ["<ip-endpoint>:9100"]
            labels:
            hostname: <server-name>
    ```

3. Validasi sintaks *file* konfigurasi Prometheus.

    ```console
    sudo promtool check config /etc/prometheus/prometheus.yml
    ```

4. *Restart* servis Prometheus.

    ```console
    sudo systemctl restart prometheus
    sudo systemctl status prometheus
    ```

## Instalasi Grafana

Berikut merupakan langkah instalasi servis Grafana.

1. Instalasi *packages* yang dibutuhkan.

    ```console
    sudo apt-get install -y apt-transport-https software-properties-common wget
    ```

2. Impor GPG *key*.

    ```console
    sudo mkdir -p /etc/apt/keyrings/
    wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
    ```

3. Tambahkan repositori untuk rilis versi stabil.

    ```console
    echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```

4. *Instal* servis Grafana.

    ```console
    sudo apt-get update
    sudo apt-get install grafana
    ```

5. *Enable* servis Grafana.

    ```console
    sudo systemctl enable grafana-server.service
    ```

    Lalu cek servisnya.

    ```console
    sudo systemctl status grafana-server.service
    ```

### Pengaturan Database

Secara *default*, Grafana menggunakan SQLite sebagai *database*-nya dan terletak di satu server yang sama. Namun, *database* tersebut kurang cocok untuk digunakan di level *production* yang mempunyai skala yang besar. Oleh karena itu, pada tutorial kali ini, saya akan menggunakan PostgreSQL atau MySQL (Pilih salah satu) sebagai *database* servis Grafana. *Database* dan servis Grafana akan berada pada server (VPS) yang berbeda. Pastikan server tempat servis Grafana dan server tempat *database* berada dalam satu *network*/ VPC agar latensi koneksinya rendah. Berikut merupakan cara instalasi dan pengaturan pada kedua *database* tersebut agar bisa terhubung ke servis Grafana.


#### Instalasi dan Pengaturan PostgreSQL

Berikut merupakan cara instalasi dan pengaturan PostgreSQL agar bisa terhubung ke servis Grafana. 

1. *Install* servis PostgreSQL melalui *command* berikut.

    ```console
    sudo apt install postgresql postgresql-contrib
    ```

2. *Start* dan *enable* servis PostgreSQL agar bisa langsung dijalankan setelah server sedang di-*reboot* atau di-*restart*.

    ```console
    sudo systemctl start postgresql.service
    sudo systemctl enable postgresql.service
    ```

    Lalu cek servisnya.

    ```console
    sudo systemctl status postgresql.service
    ```

3. *Login* ke akun *default* postgres.

    ```console
    sudo -i -u postgres
    ```

    > Akun postgres adalah akun *default* yang tercipta pertama kali ketika kita menginstalasi servis PostgreSQL.

4. Buat *user* dan *database* yang ingin digunakan di servis Grafana.

    ```console
    createuser <grafana-db-user>
    createdb <grafana-db-name> -O <grafana-db-user>
    ```

5. Masuk ke psql.

    ```console
    psql
    ```

    Pasang *password* pada *user* yang tadi dibuat.

    ```console
    ALTER USER <grafana-db-user> WITH ENCRYPTED PASSWORD '<grafana-db-password>';
    ```

6. Atur akses *remote* ke servis PostgreSQL. Pertama buka *file* postgresql.conf.

    ```console
    sudo nano /etc/postgresql/16/main/postgresql.conf
    ```

    > Secara *default*, versi servis PostgreSQL yang terpasang di Ubuntu Server 24.04 melalui `sudo apt install postgresql` adalah 16.

    Hapus *comment* pada baris yang terdapat `#listen_addresses = 'localhost'`. Ubah menjadi seperti dibawah ini:

    ```console
    ...
    listen_addresses = '*'
    ...
    ```

    Lalu buka *file* pg_hba.conf.

    ```console
    sudo nano /etc/postgresql/16/main/pg_hba.conf
    ```

    Lihat bagian di bawah konfigurasi `# IPv4 local connections:`, modifikasi bagian IP address yang bernilai `127.0.0.1/32` menjadi `0.0.0.0/0` sehingga menjadi seperti berikut.

    ```console
    ...
    # IPv4 local connections:
    host    all             all             0.0.0.0/0		scram-sha-256
    ...
    ```

7. *Restart* servis PostgreSQL.

```console
sudo systemctl restart postgresql
sudo systemctl status postgresql
```

8. Atur IP *whitelist* agar hanya server-server tertentu saja yang bisa mengakses servis PostgreSQL.

    ```console
    sudo ufw allow from <private-ip-grafana> to any port 5432
    ```

9. Atur koneksi *database* Grafana. Masuk ke *file* grafana.ini.

    ```console
    sudo nano /etc/grafana/grafana.ini
    ```

    Pada bagian `[database]`, hapus *comment* berbentuk tanda `;` pada baris berikut, lalu isi seperti di bawah ini:

    ```console
    ...
    [database]

    type = postgres
    host = <private-ip-postgresql>:5432
    name = <grafana-db-name>
    user = <grafana-db-user>

    password = <grafana-db-password>
    ...
    ```

10. *Restart* servis Grafana.

    ```console
    sudo systemctl restart grafana-server.service
    sudo systemctl status grafana-server.service
    ```

11. Masuk ke aplikasi Grafana melalui `http://<public-ip-grafana>:3000` atau melalui domain https jika Anda memasang domain untuk aplikasi Grafana. Masukkan kredensial akun sebagai berikut.

    | username | password |
    |----------|----------|
    | admin    | admin    |

#### Instalasi dan Pengaturan MySQL

Berikut merupakan cara instalasi dan pengaturan MySQL agar bisa terhubung ke servis Grafana.

1. *Install* servis MySQL melalui *command* berikut.

    ```console
    sudo apt install mysql-server
    ```

2. *Start* dan *enable* servis MySQL agar bisa langsung dijalankan setelah server sedang di-*reboot* atau di-*restart*.

    ```console
    sudo systemctl start mysql.service
    sudo systemctl enable mysql.service
    ```

    Lalu cek servisnya.

    ```console
    sudo systemctl status mysql.service
    ```

3. Jalankan *security script* instalasi servis MySQL melalui *command* berikut.

    ```console
    sudo mysql_secure_installation
    ```

    Pilih/isi semua pilihan yang ada di instruksi *prompt* sesuai kebutuhan atau *best prectice*. Pada saat memasukkan *password* akun `root`, pastikan untuk mengingatnya.

4. Masuk ke servis MySQL melalui akun root, lalu masukkan *password*.

    ```console
    sudo mysql -u root -p
    ```

5. Membuat *database* untuk servis Grafana.

    ```console
    CREATE DATABASE <grafana-db-name>;
    ```

6. Membuat akun berupa *user* dan *password* dengan batasan IP dari server tempat servis Grafana berada.

    ```console
    CREATE USER '<grafana-db-user>'@'<private-ip-grafana>' IDENTIFIED WITH mysql_native_password BY '<grafana-db-password>';
    ```

    Jika terjadi kendala karena muncul respons berikut **“Your password does not satisfy the current policy requirements”**, atur pengaturan `policy password` di MySQL menjadi `low`.

    ```console
    SET GLOBAL validate_password.policy=LOW;
    ```

7. Berikan akses *user* tersebut ke *database* Grafana yang baru dibuat.

    ```console
    GRANT ALL PRIVILEGES ON <grafana-db-name>.* TO '<grafana-db-user>'@'<private-ip-grafana>' WITH GRANT OPTION;
    ```

    *Reload* pengaturan terkait akses *database*.

    ```console
    FLUSH PRIVILEGES;
    ```

8. Konfirmasi nama *database* Grafana beserta *owner* aksesnya.

    ```console
    SELECT user,host,db FROM mysql.db;
    ```

    Lalu keluar dari servis MySQL.

    ```console
    exit
    ```

9. Atur akses *remote* ke servis MySQL. Pertama buka file /etc/mysql/mysql.conf.d/mysqld.cnf.

    ```console
    sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
    ```

    Ubah pengaturan `bind-address` dari 127.0.01 menjadi 0.0.0.0.

    ```console
    ...
    bind-address		= 0.0.0.0
    ...
    ```

10. *Restart* servis MySQL.

    ```console
    sudo systemctl start mysql.service
    ```

11. Atur IP *whitelist* agar hanya server-server tertentu saja yang bisa mengakses servis MySQL.

    ```console
    sudo ufw allow from <private-ip-grafana> to any port 3306
    ```

12. Atur koneksi *database* Grafana. Masuk ke *file* grafana.ini.

    ```console
    sudo nano /etc/grafana/grafana.ini
    ```

    Pada bagian `[database]`, hapus *comment* berbentuk tanda `;` pada baris berikut, lalu isi seperti di bawah ini:

    ```console
    ...
    [database]

    type = mysql
    host = <private-ip-mysql>:3306
    name = <grafana-db-name>
    user = <grafana-db-user>

    password = <grafana-db-password>
    ...
    ```

13. *Restart* servis Grafana.

    ```console
    sudo systemctl restart grafana-server.service
    sudo systemctl status grafana-server.service
    ```

14. Masuk ke aplikasi Grafana melalui `http://<public-ip-grafana>:3000` atau melalui domain https jika Anda memasang domain untuk aplikasi Grafana. Masukkan kredensial akun sebagai berikut.

    | username | password |
    |----------|----------|
    | admin    | admin    |

### Integrasi Prometheus ke Grafana

Untuk melakukan integrasi Prometheus ke Grafana, pastikan koneksi antar servis terhubung. Jika servis Prometheus dan Grafana berada di satu server yang sama, maka otomatis sudah terhubung. Namun, jika berada di server yang berbeda, silakan atur *firewall* atau IP *whitelist*-nya terlebih dahulu. IP *whitelist* diatur pada server tempat servis Prometheus berada. Berikut *command* yang digunakan.

```console
sudo ufw allow from *ip-grafana* to any port 9090
```

Untuk pengecekan koneksi apakah sudah terhubung atau belum, silakan cek melalui *command* berikut di server tempat servis Grafana berada.

```console
nc -zv <ip-prometheus> 9090
```

Jika terdapat kata `succeeded!`, maka koneksi antar servis berhasil terhubung.

Pada aplikasi Grafana, kita perlu melakukan integrasi ke servis Prometheus untuk melakukan visualisasi *metrics* yang tertangkap. Berikut langkah-langkahnya.

1. Masuk ke aplikasi Grafana. Lalu, pada bagian navigasi menu di sebelah kiri aplikasi Grafana, pilih **Add new connection** di bagian **Connection**.

2. *Search* dan pilih Prometheus. Lalu klik **Add new data source**.

3. Isi pengaturan **Name** dengan prometheus (silakan sesuaikan dengan keinginan). Lalu isi juga pengaturan **URL** di bagian **Connection** dengan format `http://<ip-prometheus>:9090`. Jika servis Prometheus dan servis Grafana berada di satu server, silakan gunakan format `http://localhost:9090` Biarkan sisa pengaturan lain dengan nilai *default*.

4. Klik **Save & test**. Pastikan muncul pemberitahuan “Data source successfully connected.” yang menunjukkan bahwa integrasi Prometheus ke Grafana berhasil.