# Laporan Praktikum Jaringan Komputer 2026

## Kelompok K-51

| Nama | NRP |
|---|---|
| Muhammad Hugo Rayandra E | 5027251076 |
| Arrumanta Ekna Luhkinasih | 5027251044 |

---

Topologi Jaringan & Pembagian IP

Berdasarkan ketentuan soal dan topologi GNS3:

Router (rootkit):

eth0: WAN / NAT (192.168.122.10/24)

eth1: Switch1 (10.89.5.1/24)

eth2: Switch4 (10.89.3.1/24)

eth3: Switch5 (10.89.4.1/24)

eth4: Switch6 (10.89.1.1/24)

eth5: Switch7 (10.89.2.1/24)

Switch6 (Pengamat Sayap Kiri):

alpha: 10.89.1.10 (Gateway: 10.89.1.1)

beta: 10.89.1.11 (Gateway: 10.89.1.1)

gamma: 10.89.1.12 (Gateway: 10.89.1.1)

Switch7 (Eksekutor Sayap Kanan):

delta: 10.89.2.10 (Gateway: 10.89.2.1)

epsilon: 10.89.2.11 (Gateway: 10.89.2.1)

Switch4 & Switch5 (Gerbang Penyaring):

abbey: 10.89.3.10 (Gateway: 10.89.3.1)

penny: 10.89.4.10 (Gateway: 10.89.4.1)

Switch1 → Switch2/3 (Penjaga Direktori & Repositori):

prab (Master DNS): 10.89.5.10 (Gateway: 10.89.5.1)

tedd (Slave DNS): 10.89.5.11 (Gateway: 10.89.5.1)

obladi (Vault Static 1): 10.89.5.12 (Gateway: 10.89.5.1)

desmond (Vault Static 2): 10.89.5.13 (Gateway: 10.89.5.1)

oblada (Core Dynamic 1): 10.89.5.14 (Gateway: 10.89.5.1)

molly (Core Dynamic 2): 10.89.5.15 (Gateway: 10.89.5.1)

Soal 1: Penetapan Alamat IP dan Default Gateway

Script Konfigurasi Router (rootkit)

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


Script Konfigurasi Host / Client (Non-Router)

#!/bin/bash

ip link set eth0 up
ip addr flush dev eth0
ip addr add <IP_NODE>/24 dev eth0
ip route del default 2>/dev/null
ip route add default via <GATEWAY_NODE> dev eth0

echo "nameserver 192.168.122.1" > /etc/resolv.conf

echo "SETUP HOST SELESAI"


Soal 2: Konfigurasi NAT & IP Forwarding di Router (rootkit)

Konfigurasi NAT menggunakan iptables agar seluruh subnet internal dapat terhubung ke jaringan publik melalui interface eth0.

# Aktifkan IPv4 forwarding
sysctl -w net.ipv4.ip_forward=1 >/dev/null

# Rule NAT Masquerade
iptables -t nat -C POSTROUTING -o eth0 -j MASQUERADE 2>/dev/null || \
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

iptables -C FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT 2>/dev/null || \
iptables -A FORWARD -i eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

iptables -C FORWARD -o eth0 -j ACCEPT 2>/dev/null || \
iptables -A FORWARD -o eth0 -j ACCEPT


Soal 3: Routing Internal & Resolver Awal

Setiap host non-router diarahkan untuk menambahkan resolver awal 192.168.122.1 di /etc/resolv.conf untuk kemudahan pengunduhan paket.

Pengujian koneksi internet dan DNS resolver awal:

cat /etc/resolv.conf
ping -c 3 google.com


Soal 4: Membangun DNS Authoritative Master (prab) dan Slave (tedd)

Di Node prab (Master DNS)

/etc/bind/named.conf.options:

options {
    directory "/var/cache/bind";
    recursion yes;
    forwarders {
        192.168.122.1;
    };
    allow-query { any; };
    dnssec-validation no;
};


/etc/bind/named.conf.local:

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


File Zone /etc/bind/db.K-51.com:

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


Penataan Ulang Resolver di Semua Non-Router

Setelah DNS berjalan, perbarui /etc/resolv.conf pada seluruh host non-router menjadi:

nameserver 10.89.5.10
nameserver 10.89.5.11
nameserver 192.168.122.1


Soal 5: Konfigurasi Hostname dan A Record Setiap Node

Setiap node di-set hostname-nya menggunakan script berikut:

echo "nama_node" > /etc/hostname && echo "127.0.1.1 nama_node" >> /etc/hosts && hostname -F /etc/hostname


Menambahkan A Record di DNS Master (prab):

alpha.K-51.com → 10.89.1.10

beta.K-51.com → 10.89.1.11

gamma.K-51.com → 10.89.1.12

delta.K-51.com → 10.89.2.10

epsilon.K-51.com → 10.89.2.11

abbey.K-51.com → 10.89.3.10

penny.K-51.com → 10.89.4.10

obladi.K-51.com → 10.89.5.12

desmond.K-51.com → 10.89.5.13

oblada.K-51.com → 10.89.5.14

molly.K-51.com → 10.89.5.15

Soal 6: Verifikasi Zone Transfer pada Slave (tedd)

Pengujian dilakukan untuk memastikan tedd menerima salinan zone dari prab dengan nomor serial SOA yang sama:

dig @10.89.5.10 K-51.com SOA +noall +answer
dig @10.89.5.11 K-51.com SOA +noall +answer
dig @10.89.5.11 penny.K-51.com A +noall +answer


Soal 7: Konfigurasi Domain Vault, Core, dan CNAME

Pada prab di file /etc/bind/db.K-51.com, tambahkan record berikut dan naikkan serial number:

vault       IN A       10.89.5.12
vault       IN A       10.89.5.13

core        IN A       10.89.5.14
core        IN A       10.89.5.15

www         IN CNAME   penny.K-51.com.
static      IN CNAME   abbey.K-51.com.


Jalankan perintah rndc reload untuk menerapkan perubahan.

Pengujian dari Client (alpha):

host vault.K-51.com
host core.K-51.com
host -t CNAME www.K-51.com
host -t CNAME static.K-51.com


Soal 8: Konfigurasi Reverse DNS Pointer (PTR Record)

Tambahan pada /etc/bind/named.conf.local di prab:

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


Isian File Zone Reverse Pointer:

/etc/bind/jarkom/3.89.10.in-addr.arpa:
10 IN PTR abbey.K-51.com.

/etc/bind/jarkom/4.89.10.in-addr.arpa:
10 IN PTR penny.K-51.com.

/etc/bind/jarkom/5.89.10.in-addr.arpa:
12 IN PTR vault.K-51.com.
13 IN PTR vault.K-51.com.
14 IN PTR core.K-51.com.
15 IN PTR core.K-51.com.

Soal 9: Web Server Statis Area Vault (obladi & desmond)

Membuat direktori /arsip/ dan mengaktifkan fitur autoindex pada Apache di node obladi dan desmond.

Pengujian:

curl -i http://vault.K-51.com/arsip/


Soal 10: Web Server Dinamis Area Core (oblada & molly)

Menjalankan PHP-FPM dan Nginx untuk memuat halaman beranda serta profil dengan aturan URL Rewrite (URL bersih tanpa .php).

Pengujian:

php -v
ls -lah /run/php/
ls -l /var/www/core/
curl -i http://10.89.5.14/
curl -i http://10.89.5.15/


Soal 11: Reverse Proxy (Penny & Abbey)

Penny (Apache2): Bertindak sebagai Reverse Proxy menuju area Vault (10.89.5.12 & 10.89.5.13).

Pengujian konfigurasi: apache2ctl -S / apache2ctl configtest

Memastikan directive ProxyPass dan ProxyPassReverse aktif.

Abbey (Nginx): Bertindak sebagai Reverse Proxy menuju area Core (10.89.5.14 & 10.89.5.15).

Pengujian konfigurasi: nginx -t

Soal 12: Basic Authentication di Penny

Melindungi endpoint /admin pada penny menggunakan HTTP Basic Authentication.

Pembuatan Kredensial:

htpasswd -c /etc/apache2/.htpasswd prabs
# Masukkan password: pakar_pinter_jadi_gob***


Pengujian:

Tanpa Credential (Expected: 401 Unauthorized):

curl -i http://penny.K-51.com/admin


Dengan Credential (Expected: 200 OK):

curl -i -u prabs:pakar_pinter_jadi_gob*** http://penny.K-51.com/admin


Soal 13: Konfigurasi Redirect Permanen (301) dan Sementara (302)

IP Penny / penny.K-51.com → Redirect Permanen (301) ke www.K-51.com

IP Abbey / abbey.K-51.com → Redirect Sementara (302) ke static.K-51.com

Pengujian:

curl -I http://penny.K-51.com/
curl -I http://abbey.K-51.com/


Soal 14: Konfigurasi Real Client IP

Meneruskan header IP asli pengunjung dari Reverse Proxy (X-Real-IP / X-Forwarded-For) ke backend.

Konfigurasi Nginx:

set_real_ip_from 10.89.3.10;
real_ip_header X-Real-IP;
real_ip_recursive on;


Pengujian:

tail -f /var/log/nginx/access.log
# Jalankan dari client:
curl http://static.K-51.com/


Soal 15: Konfigurasi Endpoint /eternal pada Penny

Membuat direktori /var/www/eternal dan menjalankan backend PHP pada port 127.0.1.1:8081.

Konfigurasi ProxyPass Apache:

ProxyPass        /eternal/ http://127.0.0.1:8081/
ProxyPassReverse /eternal/ http://127.0.0.1:8081/


Pengujian:

ss -lntp | grep ':8081'
curl -i http://penny.K-51.com/eternal/


Output yang diharapkan: HTTP/1.1 200 OK, X-Powered-By: PHP/8.4.x, serta konten halaman.

Soal 16: Konfigurasi Endpoint /orion pada Abbey

Membuat direktori /var/www/orion/ berisi index.html dan menyajikannya secara murni statis tanpa rendering PHP.

Pengujian:

nginx -t
curl -i http://abbey.K-51.com/orion/


Soal 17: Pengujian Performa Menggunakan ApacheBench

Pengujian dilakukan dari node alpha dengan total 250 requests dan concurrency level 10:

# Benchmark WWW
ab -n 250 -c 10 http://www.K-51.com/ > /root/ab_www.txt

# Benchmark Static
ab -n 250 -c 10 http://static.K-51.com/ > /root/ab_static.txt

# Analisis Hasil
grep -E "Complete requests|Failed requests|Requests per second|Time per request|Transfer rate" /root/ab_static.txt


Soal 18: Konfigurasi TXT Record DNS

Menambahkan TXT record untuk setiap client pengamat dan eksekutor di file zone /etc/bind/db.K-51.com:

alpha       IN TXT "alpha"
beta        IN TXT "beta"
gamma       IN TXT "gamma"
delta       IN TXT "delta"
epsilon     IN TXT "epsilon"


Pengujian:

named-checkzone K-51.com /etc/bind/db.K-51.com
dig alpha.K-51.com TXT +noall +answer
dig beta.K-51.com TXT +noall +answer


Soal 19: Pengujian TTL 15 Detik dan Cache Behavior

Menguji respon query DNS saat IP A Record abbey.K-51.com diubah secara fiktif dengan TTL 15 detik.

Fase 1 (Sebelum Perubahan):

dig abbey.K-51.com A


Fase 2 (Seketika setelah perubahan):
Memeriksa DNS query kembali. Hasil masih mengembalikan IP lama dikarenakan cache TTL belum expired.

Fase 3 (Setelah masa TTL habis / >15 detik):

sleep 16
dig abbey.K-51.com A


Hasil: Mengembalikan alamat IP fiktif yang baru.

Soal 20: Konfigurasi CNAME Outbound ke External Domain

Menambahkan record CNAME binding dari domain internal ke http.badssl.com:

outbound IN CNAME http.badssl.com.


Pengujian:

dig outbound.K-51.com CNAME +noall +answer
curl -i http://outbound.K-51.com


Hasil: Menampilkan respon HTTP/1.1 200 OK dan memuat konten dari http.badssl.com.
