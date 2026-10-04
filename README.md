# Laporan Praktikum Jaringan Komputer 2026

## Kelompok K-51

| Nama | NRP |
|---|---|
| Muhammad Hugo Rayandra E | 5027251076 |
| Arrumanta Ekna Luhkinasih | 5027251044 |

---

# Laporan Resmi Praktikum Modul 2 - Jarkom 2026
**Kelompok K-51**

## Topologi Jaringan & Pembagian IP

Berdasarkan ketentuan soal dan topologi GNS3:
* **Router (rootkit):**
  * `eth0`: WAN / NAT (192.168.122.10/24)
  * `eth1`: Switch1 (10.89.5.1/24)
  * `eth2`: Switch4 (10.89.3.1/24)
  * `eth3`: Switch5 (10.89.4.1/24)
  * `eth4`: Switch6 (10.89.1.1/24)
  * `eth5`: Switch7 (10.89.2.1/24)

* **Switch6 (Pengamat Sayap Kiri):**
  * `alpha`: 10.89.1.10 (Gateway: 10.89.1.1)
  * `beta`: 10.89.1.11 (Gateway: 10.89.1.1)
  * `gamma`: 10.89.1.12 (Gateway: 10.89.1.1)

* **Switch7 (Eksekutor Sayap Kanan):**
  * `delta`: 10.89.2.10 (Gateway: 10.89.2.1)
  * `epsilon`: 10.89.2.11 (Gateway: 10.89.2.1)

* **Switch4 & Switch5 (Gerbang Penyaring):**
  * `abbey`: 10.89.3.10 (Gateway: 10.89.3.1)
  * `penny`: 10.89.4.10 (Gateway: 10.89.4.1)

* **Switch1 → Switch2/3 (Penjaga Direktori & Repositori):**
  * `prab` (Master DNS): 10.89.5.10 (Gateway: 10.89.5.1)
  * `tedd` (Slave DNS): 10.89.5.11 (Gateway: 10.89.5.1)
  * `obladi` (Vault Static 1): 10.89.5.12 (Gateway: 10.89.5.1)
  * `desmond` (Vault Static 2): 10.89.5.13 (Gateway: 10.89.5.1)
  * `oblada` (Core Dynamic 1): 10.89.5.14 (Gateway: 10.89.5.1)
  * `molly` (Core Dynamic 2): 10.89.5.15 (Gateway: 10.89.5.1)

![Nama Gambar](path/ke/gambar.png)

---

## Soal 1: Penetapan Alamat IP dan Default Gateway

### Script Konfigurasi Router (rootkit)
```bash
#!/bin/bash

# Aktifkan interface
for iface in eth0 eth1 eth2 eth3 eth4 eth5; do
    ip link set "$iface" up
done

# WAN
ip addr replace 192.168.122.10/24 dev eth0
ip route del default 2>/dev/null
ip route add default via 192.168.122.1 dev eth0 2>/dev/null

# LAN
ip addr replace 10.89.5.1/24 dev eth1
ip addr replace 10.89.3.1/24 dev eth2
ip addr replace 10.89.4.1/24 dev eth3
ip addr replace 10.89.1.1/24 dev eth4
ip addr replace 10.89.2.1/24 dev eth5

echo "SETUP ROUTER SELESAI"
```

### Script Konfigurasi Host / Client (Non-Router)
```bash
#!/bin/bash

ip link set eth0 up
ip addr flush dev eth0
ip addr add <IP_NODE>/24 dev eth0
ip route del default 2>/dev/null
ip route add default via <GATEWAY_NODE> dev eth0

echo "nameserver 192.168.122.1" > /etc/resolv.conf

echo "SETUP HOST SELESAI"
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 2: Konfigurasi NAT & IP Forwarding di Router (rootkit)

Konfigurasi NAT menggunakan iptables agar seluruh subnet internal dapat terhubung ke jaringan publik melalui interface eth0.
```bash
# Aktifkan IPv4 forwarding
sysctl -w net.ipv4.ip_forward=1 >/dev/null

# Rule NAT Masquerade
iptables -t nat -C POSTROUTING -o eth0 -j MASQUERADE 2>/dev/null || \
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -C FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT 2>/dev/null || \
iptables -A FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

iptables -C FORWARD -o eth0 -j ACCEPT 2>/dev/null || \
iptables -A FORWARD -o eth0 -j ACCEPT
```

---

## Soal 3: Routing Internal & Resolver Awal

Setiap host non-router diarahkan untuk menambahkan resolver awal 192.168.122.1 di /etc/resolv.conf untuk kemudahan pengunduhan paket.

Pengujian koneksi internet dan DNS resolver awal:
```bash
cat /etc/resolv.conf
ping -c 3 google.com
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 4: Membangun DNS Authoritative Master (prab) dan Slave (tedd)

Di Node prab (Master DNS)
/etc/bind/named.conf.options:
```bash
options {
    directory "/var/cache/bind";
    recursion yes;
    forwarders {
        192.168.122.1;
    };
    allow-query { any; };
    dnssec-validation no;
};
```
/etc/bind/named.conf.local:
```bash
zone "K-51.com" {
    type master;
    file "/etc/bind/db.K-51.com";
    notify yes;
    also-notify {
        10.89.5.11;
    };
    allow-transfer {
        10.89.5.11;
    };
};
```
File Zone /etc/bind/db.K-51.com:
```bash
$TTL 86400
@   IN  SOA prab.K-51.com. admin.K-51.com. (
        2026092901
        3600
        1800
        604800
        86400
)

    IN  NS  prab.K-51.com.
    IN  NS  tedd.K-51.com.

@       IN  A   10.89.4.10
prab    IN  A   10.89.5.10
tedd    IN  A   10.89.5.11
```
Penataan Ulang Resolver di Semua Non-Router
Setelah DNS berjalan, perbarui /etc/resolv.conf pada seluruh host non-router menjadi:
```bash
nameserver 10.89.5.10
nameserver 10.89.5.11
nameserver 192.168.122.1
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 5: Konfigurasi Hostname dan A Record Setiap Node
Setiap node di-set hostname-nya menggunakan script berikut:
```bash
echo "nama_node" > /etc/hostname && echo "127.0.1.1 nama_node" >> /etc/hosts && hostname -F /etc/hostname
```

Menambahkan A Record di DNS Master (prab):

 - alpha.K-51.com → 10.89.1.10

 - beta.K-51.com → 10.89.1.11

 - gamma.K-51.com → 10.89.1.12

 - delta.K-51.com → 10.89.2.10

 - epsilon.K-51.com → 10.89.2.11

 - abbey.K-51.com → 10.89.3.10

 - penny.K-51.com → 10.89.4.10

 - obladi.K-51.com → 10.89.5.12

 - desmond.K-51.com → 10.89.5.13

 - oblada.K-51.com → 10.89.5.14

 - molly.K-51.com → 10.89.5.1

![Nama Gambar](path/ke/gambar.png)

---

## Soal 6: Verifikasi Zone Transfer pada Slave (tedd)
Pengujian dilakukan untuk memastikan tedd menerima salinan zone dari prab dengan nomor serial SOA yang sama:
```bash
dig @10.89.5.10 K-51.com SOA +noall +answer
dig @10.89.5.11 K-51.com SOA +noall +answer
dig @10.89.5.11 penny.K-51
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 7: Konfigurasi Domain Vault, Core, dan CNAME
Pada prab di file /etc/bind/db.K-51.com, tambahkan record berikut dan naikkan serial number:
```bash
DNS Zone file
vault       IN A       10.89.5.12
vault       IN A       10.89.5.13

core        IN A       10.89.5.14
core        IN A       10.89.5.15

www         IN CNAME   penny.K-51.com.
static      IN CNAME   abbey.K-51.com.
```
Jalankan perintah rndc reload untuk menerapkan perubahan.

Pengujian dari Client (alpha):
```bash
host vault.K-51.com
host core.K-51.com
host -t CNAME www.K-51.com
host -t CNAME static.K-51.com
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 8: Konfigurasi Reverse DNS Pointer (PTR Record)
Tambahan pada /etc/bind/named.conf.local di prab:
```bash
zone "3.89.10.in-addr.arpa" { 
    type master; 
    file "/etc/bind/jarkom/3.89.10.in-addr.arpa"; 
    notify yes; 
    also-notify { 10.89.5.11; }; 
    allow-transfer { 10.89.5.11; }; 
};

zone "4.89.10.in-addr.arpa" { 
    type master; 
    file "/etc/bind/jarkom/4.89.10.in-addr.arpa"; 
    notify yes; 
    also-notify { 10.89.5.11; }; 
    allow-transfer { 10.89.5.11; }; 
};

zone "5.89.10.in-addr.arpa" { 
    type master; 
    file "/etc/bind/jarkom/5.89.10.in-addr.arpa"; 
    notify yes; 
    also-notify { 10.89.5.11; }; 
    allow-transfer { 10.89.5.11; }; 
};
```

Isian File Zone Reverse Pointer:
- /etc/bind/jarkom/3.89.10.in-addr.arpa:
  10 IN PTR abbey.K-51.com.

- /etc/bind/jarkom/4.89.10.in-addr.arpa:
  10 IN PTR penny.K-51.com.

- /etc/bind/jarkom/5.89.10.in-addr.arpa:
  12 IN PTR vault.K-51.com.
  13 IN PTR vault.K-51.com.
  14 IN PTR core.K-51.com.
  15 IN PTR core.K-51.com.

![Nama Gambar](path/ke/gambar.png)

---

## Soal 9: Web Server Statis Area Vault (obladi & desmond)
Membuat direktori /arsip/ dan mengaktifkan fitur autoindex pada Apache di node obladi dan desmond.

Pengujian:
```bash
curl -i http://vault.K-51.com/arsip/
```

![Nama Gambar](path/ke/gambar.png)

---

## Soal 10: Web Server Dinamis Area Core (oblada & molly)
Menjalankan PHP-FPM dan Nginx untuk memuat halaman beranda serta profil dengan aturan URL Rewrite (URL bersih tanpa .php).

Pengujian:
```bash
php -v
ls -lah /run/php/
ls -l /var/www/core/
curl -i http://10.89.5.14/
curl -i http://10.89.5.15/
```

## 11. Konfigurasi Reverse Proxy

### 11.1 Reverse Proxy Penny

Penny menggunakan Apache sebagai reverse proxy.

Module yang dibutuhkan diaktifkan:
```bash
a2enmod proxy
a2enmod proxy_http
a2enmod headers
```
Konfigurasi kemudian diperiksa menggunakan:
```
apache2ctl configtest
```
Hasil yang diharapkan:
```
Syntax OK
```
### 11.2 Reverse Proxy Abbey

Abbey menggunakan Nginx sebagai reverse proxy.

Konfigurasi diperiksa menggunakan:
```
nginx -t
```
Hasil yang diharapkan:
```
syntax is ok
test is successful
```
## 12. Konfigurasi Basic Authentication

Authentication digunakan untuk membatasi akses menuju endpoint tertentu.

File password dibuat menggunakan:
```
htpasswd -c /etc/apache2/.htpasswd <username>
```
Konfigurasi Apache menggunakan:
```
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```
Kemudian dilakukan pengujian tanpa credential:
```
curl -i http://penny.K-51.com/admin
```
Response yang diharapkan:
```
401 Unauthorized
```
Setelah itu dilakukan pengujian menggunakan credential:
```
curl -i -u <username>:<password> http://penny.K-51.com/admin
```
## 13. Konfigurasi Redirect

Redirect diuji menggunakan opsi -I pada curl agar hanya header HTTP yang ditampilkan.
```
curl -I http://penny.K-51.com/

dan:

curl -I http://abbey.K-51.com/
```
Pada hasil pengujian diperiksa:

HTTP/1.1 301

atau:

HTTP/1.1 302

serta header:

Location:

Header tersebut menunjukkan tujuan redirect yang telah dikonfigurasi.

## 14. Konfigurasi Real Client IP

Pada reverse proxy dilakukan konfigurasi agar alamat IP client asli dapat diteruskan menuju backend.

Contoh konfigurasi Nginx:
```
set_real_ip_from 10.89.3.10;
real_ip_header X-Real-IP;
real_ip_recursive on;
```
Kemudian log Nginx diamati:
```
tail -f /var/log/nginx/access.log
```
Dari client dilakukan request:
```
curl http://static.K-51.com/
```
Log kemudian diperiksa untuk memastikan informasi alamat client diterima oleh server.

## 15. Konfigurasi Eternal pada Penny

Directory Eternal dibuat pada Penny:
```
mkdir -p /var/www/eternal
```
Kemudian dibuat file PHP:
```
/var/www/eternal/index.php
```
Backend PHP dijalankan pada port:
```
127.0.0.1:8081
```
Port diperiksa menggunakan:
```
ss -lntp | grep ':8081'
```
Pada proses pengerjaan sempat ditemukan error:

Address already in use

Error tersebut terjadi karena port 8081 sudah digunakan oleh proses PHP yang telah berjalan.

Untuk memastikan proses yang menggunakan port tersebut:
```
ss -lntp | grep ':8081'
```
Jika proses PHP sudah terlihat, backend tidak perlu dijalankan kembali.

Konfigurasi Apache kemudian menggunakan:
```
ProxyPass        /eternal/ http://127.0.0.1:8081/
ProxyPassReverse /eternal/ http://127.0.0.1:8081/
```
Validasi Apache:
```
apache2ctl configtest
```
Kemudian dilakukan pengujian:
```
curl -i http://penny.K-51.com/eternal/
```
Hasil pengujian yang diperoleh:
```
HTTP/1.1 200 OK
X-Powered-By: PHP/8.4.26
```
Response juga menampilkan:

ETERNAL - PENNY
PHP STATUS: AKTIF
Hostname: penny

Hasil tersebut menunjukkan bahwa request berhasil melewati reverse proxy Penny dan PHP berhasil dirender.

## 16. Konfigurasi Orion pada Abbey

Directory Orion dibuat sebagai static content:
```
/var/www/orion/
```
Kemudian dibuat:

index.html

Konfigurasi Nginx digunakan untuk melayani endpoint:

/orion/

Konfigurasi diuji menggunakan:
```
nginx -t
```
Setelah konfigurasi valid, Nginx direload.

Pengujian dilakukan:
```
curl -i http://abbey.K-51.com/orion/
```
Endpoint Orion harus menghasilkan static content dan tidak menjalankan PHP.

## 17. Pengujian ApacheBench

Pengujian performa dilakukan dari Alpha menggunakan ApacheBench.

Untuk domain www:
```
ab -n 250 -c 10 http://www.K-51.com/ > /root/ab_www.txt
```
Untuk domain static:
```
ab -n 250 -c 10 http://static.K-51.com/ > /root/ab_static.txt
```
Hasil kemudian diperiksa:
```
grep -E \
"Complete requests|Failed requests|Requests per second|Time per request|Transfer rate" \
/root/ab_static.txt
```
Salah satu hasil pengujian yang diperoleh:
```
Complete requests:      250
Failed requests:        124
Requests per second:    1699.99 [#/sec]
Time per request:       5.882 [ms]
Transfer rate:          244.86 [Kbytes/sec]
```
Pada bagian failed requests ditemukan:
```
Connect: 0
Receive: 0
Length: 124
Exceptions: 0
```
Nilai tersebut menunjukkan bahwa kegagalan yang terdeteksi ApacheBench berada pada perbedaan panjang response, bukan kegagalan koneksi atau exception.

18. Konfigurasi TXT Record

TXT record ditambahkan ke DNS zone.

Record yang dibuat:
```
alpha       IN TXT "alpha"
beta        IN TXT "beta"
gamma       IN TXT "gamma"
delta       IN TXT "delta"
epsilon     IN TXT "epsilon"
```
Setelah penambahan record dilakukan validasi:
```
named-checkzone K-51.com /etc/bind/db.K-51.com
```
Kemudian dilakukan query:
```
dig alpha.K-51.com TXT +noall +answer
dig beta.K-51.com TXT +noall +answer
dig gamma.K-51.com TXT +noall +answer
dig delta.K-51.com TXT +noall +answer
dig epsilon.K-51.com TXT +noall +answer
```
Hasil query digunakan sebagai bukti bahwa TXT record berhasil dibuat dan dapat di-resolve.

19. Pengujian TTL 15 Detik

Sebelum melakukan perubahan TTL, file zone dibuat backup:
```
cp /etc/bind/db.K-51.com \
/etc/bind/db.K-51.com.bak-nomor18
```
TTL record kemudian diatur menjadi:

15

Pengujian dilakukan dalam tiga fase.

Fase 1

Record diperiksa sebelum perubahan:
```
dig abbey.K-51.com A
```
Fase 2

Record diubah dan segera dilakukan query kembali.

Tahap ini digunakan untuk melihat kondisi cache sebelum TTL habis.

Fase 3

Setelah menunggu TTL:

sleep 16

Kemudian query kembali:
```
dig abbey.K-51.com A
```
Waktu 16 detik digunakan sebagai waktu tunggu sedikit lebih lama dari TTL 15 detik sehingga cache diharapkan sudah expired.

Setelah pengujian selesai, konfigurasi DNS dikembalikan ke kondisi normal.

20. Konfigurasi CNAME Outbound

Record berikut ditambahkan:

outbound IN CNAME http.badssl.com.

Konfigurasi divalidasi:

named-checkzone K-51.com /etc/bind/db.K-51.com

Hasil:

OK

Kemudian dilakukan verifikasi dari Alpha:

dig outbound.K-51.com CNAME +noall +answer

Hasil yang diperoleh:

outbound.K-51.com. 86400 IN CNAME http.badssl.com.

Setelah DNS berhasil di-resolve, dilakukan pengujian HTTP:

curl -i http://outbound.K-51.com

Hasil pengujian mendapatkan:

HTTP/1.1 200 OK

Hal ini menunjukkan bahwa hostname outbound.K-51.com berhasil di-resolve sebagai CNAME dan request HTTP mendapatkan response dari tujuan tersebut.

21. Final Verification

Setelah seluruh konfigurasi selesai, dilakukan pemeriksaan akhir.

DNS Master

named-checkzone K-51.com /etc/bind/db.K-51.com

DNS Slave

dig @10.89.5.11 K-51.com SOA

Apache

apache2ctl configtest

Nginx

nginx -t

PHP-FPM

ls -lah /run/php/

DNS Resolution

getent hosts www.K-51.com
getent hosts static.K-51.com
getent hosts penny.K-51.com
getent hosts abbey.K-51.com
getent hosts outbound.K-51.com

Web Testing

curl -I http://www.K-51.com/
curl -I http://static.K-51.com/
curl -i http://penny.K-51.com/eternal/
curl -i http://abbey.K-51.com/orion/
curl -i http://outbound.K-51.com/

Seluruh hasil pengujian tersebut digunakan sebagai verifikasi akhir bahwa konfigurasi DNS, re

![Nama Gambar](path/ke/gambar.png)

---

