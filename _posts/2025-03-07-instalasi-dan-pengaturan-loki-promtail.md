---
title: Instalasi dan Pengaturan Loki dan Promtail
date: 2025-03-07 23:59:59 +0700
categories: [DevOps]
tags: [devops, devops tools, logging system, loki, promtail]
---

## Arsitektur Sistem pada Project

Implementasi *projects* yang akan dibangun menggunakan *existing infrastructure* atau infrastruktur yang sudah ada. Berikut merupakan gambaran dari arsitektur dari infrastruktur sistem yang ada.

![project-loki](/assets/img/posts/devops/arsitektur-project-loki.png)
_Arsitektur Project Loki_

*Projects* ini merupakan *VM-based*, jadi tidak menggunakan Docker atau *container engine* lain dalam penerapannya. Setiap satu ikon merepresentasikan sebuah server atau *node* yang berupa VM (*Virtual Machine*).

> Walaupun *project*-nya *VM-based*, harusnya tidak akan ada perbedaan yang signifikan bila coba diterapkan dengan menggunakan Docker.

## Informasi Lingkungan Project

Semua sistem operasi yang terdapat pada VM yang ada menggunakan Ubuntu LTS versi 22.04. Berikut beberapa informasi terkait lingkungan perangkat lunak *projects* yang digunakan.

| Sistem | Nama Aplikasi/*Tools*/dsb | Deskripsi | Versi |
|---|---|---|---|
| *Database* (Server Target) | MongoDB Community Edition | Server target untuk mengirim log *slow query* MongoDB ke Loki. | 4.4.x |
| Reverse Proxy (Server Target) | Nginx | Server target untuk mengirim log *web* server Nginx ke Loki. | 1.18.0 |
| *Logging* | Loki | *Tool* untuk sistem *logging* terpusat. | 3.0.0 |
| *Log Agent* | Promtail | *Tool* untuk mengirim log dari server target ke Loki. | 3.0.0 |
| *Object Storage* (S3-Compatible) | IDCloudHost | *Service* untuk menyimpan log secara terpusat. | - |

Pastikan sistem di atas sudah tersedia sebelum mengikuti instruksi-instruksi selanjutnya. Alternatif untuk IDCloudHost *object storage* bisa menggunakan alternatif *object storage* lain selama *S3-compatible*, beberapa diantaranya dapat menggunakan Minio, DigitalOcean Space, atau AWS S3. *Project* ini juga bisa dijalankan menggunakan diska bawaan dari VM/VPS tanpa memerlukan *object storage*.

## Instalasi Loki
Berikut merupakan langkah-langkah proses instalasi *service* Loki.

1. Buat *system user* untuk menjalankan *service* Loki.

```console
sudo useradd --system loki
```

2. Unduh *file* Loki. Sesuai informasi di atas, versi yang digunakan adalah 3.0.0.

```console
wget https://github.com/grafana/loki/releases/download/v3.0.0/loki-linux-amd64.zip
```

3. Ekstrak *file* Loki yang barusan diunduh.

```console
unzip loki-linux-amd64.zip
```

4. Salin *file* binari Loki sebelumnya ke direktori /usr/local/bin/.

```console
sudo cp loki-linux-amd64 /usr/local/bin/loki
```

5. Buat *file* pengaturan *service* Loki sebagai systemd.

```console
sudo nano /etc/systemd/system/loki.service
```

6. Isi *file* tersebut dengan *script* berikut.

```console
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=loki
ExecStart=/usr/local/bin/loki -config.file /etc/loki/loki-local-config.yaml

[Install]
WantedBy=multi-user.target
```

7. Buat direktori untuk menyimpan *file* pengaturan Loki.

```console
mkdir -p /etc/loki
cd /etc/loki
```

8. Unduh *file* pengaturan *default* Loki.

```console
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/cmd/loki/loki-local-config.yaml
```

9. Buat direktori untuk penyimpanan data *service* Loki.

```console
cd
sudo mkdir -p /loki
```

10. Atur *permission file* maupun direktori terkait *service* Loki.

```console
sudo chown loki:loki /usr/local/bin/loki
sudo chown -R loki:loki /loki /etc/loki
```

11. *Reload* pengaturan systemd. Start lalu enable *service* Loki.

```console
sudo systemctl daemon-reload
sudo systemctl start loki.service
sudo systemctl enable loki.service
sudo systemctl status loki.service
```

12. Atur agar *service* Loki hanya bisa diakses melalui Grafana. Dalam kasus ini, server menggunakan pengaturan *firewall* ufw.

```console
sudo ufw allow from <ip-address> to any port 3100
```

> Gunakan IP *private* dari server Grafana.

## File Pengaturan Loki

Buka *file* pengaturan *default* Loki yang diunduh sebelumnya.

```console
sudo nano /etc/loki/loki-local-config.yaml
```

Lalu isi dan sesuaikan dengan *script* dibawah ini.

```console

# This is a complete configuration to deploy Loki backed by a s3-compatible API
# like MinIO for storage.
# Index files will be written locally at /loki/index and, eventually, will be shipped to the storage via tsdb-shipper.

auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /loki

schema_config:
  configs:
  - from: 2020-05-15
    store: tsdb
    object_store: s3
    schema: v13
    index:
      prefix: index_
      period: 24h

storage_config:
 tsdb_shipper:
   active_index_directory: /loki/index
   cache_location: /loki/index_cache
 aws:
   bucketnames: <nama-bucket>
   endpoint: is3.cloudhost.id
   access_key_id: <access-key>
   secret_access_key: <secret-key>
   s3forcepathstyle: true
 filesystem:
   directory: /loki/chunks

limits_config:
  max_query_lookback: 168h # 7 days
  retention_period: 168h
  max_query_series: 10000000
  max_query_parallelism: 2

compactor:
  working_directory: /data/retention
  retention_enabled: true
  delete_request_store: s3

ruler:
  alertmanager_url: http://localhost:9093
```

**Keterangan:**

Berikut beberapa keterangan pengaturan dari isi *script* di atas.
- **http_listen_port**: *Port* yang digunakan untuk menajalankan *service* Loki di server.
- **path_prefix**: Direktori penyimpanan data terkait *service* Loki di server.
- **schema_config** -> **configs** -> **object_store**: Tipe penyimpanan log pada Loki. Filesystem menggunakan diska pada server VM Loki terinstal. S3 menggunakan *object storage* AWS S3 maupun selain AWS S3 tapi *S3-compatible*.
- **storage_config** -> **aws** -> **bucketnames**: Nama *bucket object storage*.
- **storage_config** -> **aws** -> **endpoint**: Nama *endpoint* dari *bucket object storage* yang dibuat.
- **storage_config** -> **aws** -> **access_key_id**: *Access key* dari *bucket object storage* yang dibuat.
- **storage_config** -> **aws** -> **secret-access-key**: *Secret key* dari *bucket object storage* yang dibuat.
- **limits_config** -> **retention_period**: Lama retensi *files* log disimpan di *object storage*.
- **limits_config** -> **max_query_series**: Banyak log yang bisa diambil dalam satu kueri.
- **compactor** -> **retention_enabled**: Mengaktifkan masa retensi penyimpanan log.

## Instalasi Promtail
Berikut merupakan langkah-langkah proses instalasi *service* Promtail.
> Pastikan *service* Promtail diinstal di server target, dalam kasus ini diinstal di dalam server yang terdapat *service* Nginx maupun MongoDB.

1. Unduh *file* Promtail. Versi yang digunakan adalah 3.0.0.

```console
wget https://github.com/grafana/loki/releases/download/v3.0.0/promtail-linux-amd64.zip
```

2. Ekstrak *file* Promtail yang barusan diunduh.

3. Salin *file* binari Promtail sebelumnya ke direktori /usr/local/bin/.

4. Buat *file* pengaturan *service* Promtail sebagai systemd.

```console
sudo nano /etc/systemd/system/promtail.service
```

5. Isi *file* tersebut dengan *script* berikut.

```console
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail/promtail-local-config.yaml

[Install]
WantedBy=multi-user.target
```

6. Buat direktori untuk menyimpan *file* pengaturan Promtail.

```console
mkdir -p /etc/promtail
cd /etc/promtail
```

7. Unduh *file* pengaturan *default* Promtail.

```console
wget https://raw.githubusercontent.com/grafana/loki/v3.0.0/clients/cmd/promtail/promtail-local-config.yaml
```

8. *Reload* pengaturan systemd. Start lalu enable *service* Promtail.

```console
sudo systemctl daemon-reload
sudo systemctl start promtail.service
sudo systemctl enable promtail.service
sudo systemctl status promtail.service
```

9. Atur agar *service* Promtail hanya bisa diakses melalui Loki. Dalam kasus ini, server menggunakan pengaturan *firewall* ufw.

```console
sudo ufw allow from <ip-address> to any port 3100
```

> Gunakan IP *private* dari server Loki.