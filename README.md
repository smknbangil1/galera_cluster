# galera_cluster dgn xtrabackup
### Menggunakan SST dengan `xtrabackup`
pastikan mariadb sudah running dgn baik
1. menginstal `percona-xtrabackup`:
   ```bash
   apt install -y percona-xtrabackup
   ```
2. Ubah metode SST menjadi `xtrabackup`:
   ```ini
   wsrep_sst_method = xtrabackup
   wsrep_sst_auth = "user:password"  # Ganti dengan username/password Anda
   ```
Konfig Galera
```
[galera]
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address=gcomm://
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_cluster_name="db_cluster"
wsrep_node_address="192.168.0.11"
wsrep_slave_threads = 28
wsrep_provider_options="gcs.fc_limit=140; gcs.fc_factor=0.8"
wsrep_notify_cmd = notify
wsrep_sst_method = xtrabackup
```

---
Kesalahan ini menunjukkan bahwa MariaDB mencoba menjalankan perintah `notify`, tetapi perintah tersebut tidak ditemukan di sistem Anda. Kesalahan ini sering terkait dengan parameter `wsrep_notify_cmd` di konfigurasi MariaDB yang mengatur perintah untuk notifikasi status Galera Cluster.

Berikut langkah-langkah untuk mengatasi masalah ini:

---

### 1. **Periksa Konfigurasi MariaDB**
Cek konfigurasi `wsrep_notify_cmd` di file konfigurasi MariaDB.

1. Buka file konfigurasi MariaDB:
   ```bash
   sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
   ```
2. Cari entri `wsrep_notify_cmd`. Contohnya:
   ```ini
   wsrep_notify_cmd = notify
   ```
3. Jika ada, ini menunjukkan bahwa MariaDB mencoba menjalankan perintah bernama `notify`.

---

### 2. **Nonaktifkan Sementara `wsrep_notify_cmd`**
Jika Anda tidak memerlukan notifikasi, nonaktifkan sementara parameter ini.

1. Komentari atau hapus parameter tersebut:
   ```ini
   # wsrep_notify_cmd = notify
   ```
2. Simpan file dan restart MariaDB:
   ```bash
   sudo systemctl restart mariadb
   ```

---

### 3. **Pastikan Perintah `notify` Tersedia**
Jika Anda memerlukan fitur notifikasi, pastikan perintah `notify` atau skrip yang dirujuk tersedia.

1. **Periksa apakah file `notify` ada di sistem**:
   ```bash
   which notify
   ```
   Jika hasilnya kosong, berarti perintah tidak ada.

2. **Buat Skrip `notify`**:
   Jika Anda memerlukan notifikasi, buat skrip `notify`. Contoh sederhana:
   ```bash
   sudo nano /usr/local/bin/notify
   ```
   Isi skrip dengan konten seperti ini:
   ```bash
   #!/bin/bash
   echo "$(date) - Status: $@" >> /var/log/wsrep_notify.log
   ```
   Simpan file, lalu ubah izin agar dapat dieksekusi:
   ```bash
   sudo chmod +x /usr/local/bin/notify
   ```

3. **Update Konfigurasi MariaDB**:
   Ubah `wsrep_notify_cmd` agar merujuk ke skrip tersebut:
   ```ini
   wsrep_notify_cmd = /usr/local/bin/notify
   ```

---

### 4. **Restart MariaDB**
Setelah semua langkah selesai, restart MariaDB untuk menerapkan perubahan:
```bash
sudo systemctl restart mariadb
```

---

### 5. **Cek Log**
Periksa apakah kesalahan masih terjadi:
```bash
sudo journalctl -u mariadb | grep ERROR
```

Jika tidak ada kesalahan lagi, konfigurasi Anda sudah berhasil diperbaiki. Jika Anda menggunakan skrip notifikasi, Anda dapat memeriksa file log untuk memastikan skrip berfungsi dengan baik:
```bash
cat /var/log/wsrep_notify.log
```

---

### 6. **Langkah Opsional**
Jika Anda ingin membuat notifikasi yang lebih kompleks (misalnya, mengirim email atau webhook), modifikasi skrip `notify` sesuai kebutuhan Anda.
