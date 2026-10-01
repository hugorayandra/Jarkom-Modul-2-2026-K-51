# Jarkom-Modul-2-2026-K-51

### Shadow Net Operation — The Mesh

---

## Identitas Kelompok K-51

| No. | Nama                         |     NRP    |
| :-: | ---------------------------- | :--------: |
|  1  | Arrumanta Ekna Luhkinasih | 5027251044 |
|  2  | Muhammad Hugo Rayandra E   | 5027251076 |

# Daftar Isi

- [1. Deskripsi Praktikum](#1-deskripsi-praktikum)
- [2. Topologi dan Arsitektur](#2-topologi-dan-arsitektur)
- [3. Glosarium Entitas](#3-glosarium-entitas)
- [4. Struktur Repository](#4-struktur-repository)
- [5. Persiapan Environment](#5-persiapan-environment)
- [6. Konfigurasi Jaringan](#6-konfigurasi-jaringan)
  - [Soal 1 — IP Address dan Gateway](#soal-1--ip-address-dan-gateway)
  - [Soal 2 — WAN dan NAT](#soal-2--wan-dan-nat)
  - [Soal 3 — Routing Internal dan Resolver](#soal-3--routing-internal-dan-resolver)
- [7. Konfigurasi DNS](#7-konfigurasi-dns)
  - [Soal 4 — DNS Master](#soal-4--dns-master)
  - [Soal 5 — Hostname](#soal-5--hostname)
  - [Soal 6 — Zone Transfer](#soal-6--zone-transfer)
  - [Soal 7 — A Record dan CNAME](#soal-7--a-record-dan-cname)
  - [Soal 8 — Reverse DNS](#soal-8--reverse-dns)
- [8. Web Server](#8-web-server)
  - [Soal 9 — Apache Static Web](#soal-9--apache-static-web)
  - [Soal 10 — Nginx PHP-FPM](#soal-10--nginx-php-fpm)
- [9. Reverse Proxy](#9-reverse-proxy)
  - [Soal 11 — Reverse Proxy](#soal-11--reverse-proxy)
  - [Soal 12 — Basic Authentication](#soal-12--basic-authentication)
  - [Soal 13 — HTTP Redirect](#soal-13--http-redirect)
  - [Soal 14 — Client IP Logging](#soal-14--client-ip-logging)
  - [Soal 15 — Path Proxy](#soal-15--path-proxy)
- [10. Testing](#10-testing)
  - [Soal 16 — ApacheBench](#soal-16--apachebench)
  - [Soal 17 — TXT Record](#soal-17--txt-record)
  - [Soal 18 — DNS TTL](#soal-18--dns-ttl)
  - [Soal 19 — External CNAME](#soal-19--external-cname)
- [11. Persistence](#11-persistence)
  - [Soal 20 — Autostart dan Persistence](#soal-20--autostart-dan-persistence)
- [12. Alur Keseluruhan](#12-alur-keseluruhan)
- [13. Verifikasi Akhir](#13-verifikasi-akhir)
- [14. Troubleshooting](#14-troubleshooting)
- [15. Persiapan Demo](#15-persiapan-demo)
- [16. Checklist](#16-checklist)

---

# 1. Deskripsi Praktikum

Praktikum Modul 2 membangun sebuah jaringan terisolasi bernama **The Mesh**.

Pada topologi ini terdapat:

- `rootkit` sebagai router/gateway utama
- `alpha`, `beta`, `gamma` sebagai client sayap kiri
- `delta`, `epsilon` sebagai client sayap kanan
- `prab`, `tedd` sebagai DNS server
- `abbey`, `penny` sebagai reverse proxy
- `obladi`, `desmond` sebagai web server statis
- `oblada`, `molly` sebagai web server dinamis

Praktikum mencakup:

1. Konfigurasi IP
2. Routing
3. NAT
4. DNS authoritative
5. DNS master-slave
6. Zone transfer
7. Reverse DNS
8. Apache
9. Nginx
10. PHP-FPM
11. Reverse proxy
12. Basic authentication
13. HTTP redirect
14. Access logging
15. ApacheBench
16. DNS TXT
17. DNS TTL dan caching
18. CNAME
19. Persistence dan autostart

---

# 2. Topologi dan Arsitektur

## 2.1 Gambaran Umum

```text
                           INTERNET
                              |
                              |
                             WAN
                              |
                       +--------------+
                       |    rootkit   |
                       |    ROUTER    |
                       +--------------+
                         /    |    \
                        /     |     \
                       /      |      \
                  Network 1 Network 2 Network 3
                     |        |        |
                  Clients     DNS     Proxy
                              |        |
                           prab/tedd  |
                                      |
                              +-------+-------+
                              |               |
                            penny           abbey
                           Apache           Nginx
                              |               |
                         +----+----+      +----+----+
                         |         |      |         |
                      obladi    desmond  oblada    molly
                       STATIC     STATIC  DYNAMIC  DYNAMIC

3. Glosarium Entitas
Host	Fungsi
rootkit	Router sentral / gateway
alpha	Client
beta	Client
gamma	Client
delta	Client
epsilon	Client
prab	DNS Master
tedd	DNS Slave
penny	Apache Reverse Proxy
abbey	Nginx Reverse Proxy
obladi	Web Static
desmond	Web Static
oblada	Web Dynamic
molly	Web Dynamic


4. Informasi Addressing
Isi bagian ini sesuai konfigurasi kelompok.

4.1 Prefix Kelompok
Prefix:
10.89.x.x

4.2 Tabel IP Address
Host	Interface	IP Address	Prefix	Gateway
rootkit	ethX	XX.XX.XX.XX	/XX	-
alpha	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
beta	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
gamma	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
delta	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
epsilon	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
prab	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
tedd	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
abbey	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
penny	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
obladi	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
desmond	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
oblada	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX
molly	eth0	XX.XX.XX.XX	/XX	XX.XX.XX.XX


5. Struktur Repository
Contoh struktur repository:
.
├── README.md
│
├── GNS3/
│   └── project-files/
│
├── scripts/
│   ├── rootkit/
│   ├── prab/
│   ├── tedd/
│   ├── penny/
│   ├── abbey/
│   ├── obladi/
│   ├── desmond/
│   ├── oblada/
│   └── molly/
│
├── configs/
│   ├── dns/
│   ├── apache/
│   ├── nginx/
│   └── network/
│
├── screenshots/
│   ├── soal-01/
│   ├── soal-02/
│   ├── soal-03/
│   ├── ...
│   └── soal-20/
│
└── documentation/
    ├── troubleshooting.md
    └── demo-notes.md

6. Persiapan Environment
Praktikum menggunakan:
- GNS3 Web/Client/VM
- Docker image yang ditentukan praktikum
- Alpine/Debian sesuai kebutuhan node
- Wireshark jika diperlukan untuk observasi traffic
Image yang digunakan:
ardhptr21/alpinet:latest

atau:
ardhptr21/debinet:latest

7. Konfigurasi Jaringan
Soal 1 — IP Address dan Gateway
Tujuan
Menghubungkan rootkit dengan lima jaringan dan memberikan IP address serta default gateway kepada seluruh entitas.
Konsep
Konfigurasi yang dibutuhkan:
IP Address
Subnet
Default Gateway
Routing

Alur
Host
  |
  | packet
  v
Default Gateway
  |
  v
rootkit
  |
  v
Network tujuan

Konfigurasi
rootkit
ip addr
ip route

Tambahkan konfigurasi interface sesuai topologi:
# CONTOH
ip addr add <IP>/<PREFIX> dev <INTERFACE>
ip link set <INTERFACE> up

Host
ip addr
ip route

Set gateway:
ip route add default via <GATEWAY>

Verifikasi
ip addr
ip route

Tes:
ping <GATEWAY>
ping <HOST-LAIN>

Hasil
- [ ] Semua interface aktif
- [ ] Semua host mempunyai IP
- [ ] Default gateway benar
- [ ] Host dapat berkomunikasi
Dokumentasi
  
Soal 2 — WAN dan NAT
Tujuan
Membuat jaringan internal dapat mengakses jaringan luar melalui rootkit.
Konsep
Internal Network
       |
       v
    rootkit
       |
      NAT
       |
       v
   WAN / Internet

IP Forwarding
Periksa:
cat /proc/sys/net/ipv4/ip_forward

Aktifkan jika diperlukan:
sysctl -w net.ipv4.ip_forward=1

NAT
Konfigurasi NAT sesuai environment praktikum.
Contoh konsep:
iptables -t nat -A POSTROUTING -o <WAN_INTERFACE> -j MASQUERADE

Verifikasi
ip route

Tes dari client:
ping 192.168.122.1

Hasil
- [ ] WAN aktif
- [ ] IP forwarding aktif
- [ ] NAT aktif
- [ ] Client dapat mengakses jaringan luar
Dokumentasi
 
Soal 3 — Routing Internal dan Resolver
Tujuan
Memastikan seluruh host dapat berkomunikasi dan resolver awal tersedia.
Resolver Awal
192.168.122.1

Konfigurasi:
cat /etc/resolv.conf

Contoh:
nameserver 192.168.122.1

Verifikasi Routing
ip route

Tes:
ping <HOST-LAIN>

Verifikasi DNS
nslookup google.com
```

atau:
dig google.com

8. Konfigurasi DNS
Soal 4 — DNS Master dan Slave
Tujuan
Membangun:
prab = DNS Master
tedd = DNS Slave

Zone:
<xxxx>.com

Arsitektur
             <xxxx>.com
                  |
          +-------+-------+
          |               |
        prab             tedd
       MASTER            SLAVE
          |
          | Zone Transfer
          v
         tedd

Record yang diperlukan
SOA
<xxxx>.com. IN SOA prab.<xxxx>.com. ...

NS
<xxxx>.com. IN NS prab.<xxxx>.com.
<xxxx>.com. IN NS tedd.<xxxx>.com.

A
prab.<xxxx>.com  -> <IP-PRAB>
tedd.<xxxx>.com  -> <IP-TEDD>
<xxxx>.com       -> <IP-PENNY>

Forwarder
192.168.122.1

Verifikasi
dig SOA <xxxx>.com
dig NS <xxxx>.com
dig A prab.<xxxx>.com
dig A tedd.<xxxx>.com

Soal 5 — Hostname
Tujuan
Memberikan hostname sesuai glosarium.
Daftar Hostname
rootkit
alpha
beta
gamma
delta
epsilon
prab
tedd
abbey
penny
obladi
desmond
oblada
molly

Verifikasi
hostname
hostnamectl

DNS
Contoh:
alpha.<xxxx>.com
beta.<xxxx>.com
gamma.<xxxx>.com

Soal 6 — Zone Transfer
Tujuan
Memastikan tedd mempunyai salinan zone terbaru dari prab.
Konsep
prab
MASTER
 |
 | AXFR / IXFR
 v
tedd
SLAVE

Verifikasi SOA
Pada prab:
dig SOA <xxxx>.com

Pada tedd:
dig SOA <xxxx>.com

Serial harus sama:
PRAB  : XXXXX
TEDD  : XXXXX

Hasil
- [ ] Zone transfer berhasil
- [ ] Serial master dan slave sama
- [ ] Tedd authoritative
Soal 7 — A Record dan CNAME
A Record
vault.<xxxx>.com

mengarah ke:
obladi
desmond

core.<xxxx>.com

mengarah ke:
oblada
molly

CNAME
www.<xxxx>.com
        ↓
penny.<xxxx>.com

static.<xxxx>.com
        ↓
abbey.<xxxx>.com

Verifikasi
dig A vault.<xxxx>.com
dig A core.<xxxx>.com
dig CNAME www.<xxxx>.com
dig CNAME static.<xxxx>.com

Soal 8 — Reverse DNS
Tujuan
Membuat:
IP
 ↓
hostname

menggunakan PTR.
Target
abbey
penny
obladi
desmond
oblada
molly

Verifikasi
dig -x <IP>

Contoh:
dig -x <IP-ABBEY>

Hasil:
<IP>
 ↓
abbey.<xxxx>.com

9. Web Server
Soal 9 — Apache Static Web
Target
obladi
desmond

menggunakan:
Apache

Directory
/arsip/

Tujuan
Mengaktifkan:
Directory Listing / Autoindex

Verifikasi
curl http://<hostname>/arsip/

atau menggunakan browser.
Checklist
- [ ] Apache aktif
- [ ] /arsip/ tersedia
- [ ] Directory listing aktif
- [ ] Akses menggunakan hostname
- [ ] Obladi berhasil
- [ ] Desmond berhasil
Soal 10 — Nginx + PHP-FPM
Target
oblada
molly

menggunakan:
Nginx
PHP-FPM

Arsitektur
Client
  |
  v
Nginx
  |
  | FastCGI
  v
PHP-FPM
  |
  v
PHP Application

Halaman
/

dan:
/profil

Rewrite
URL:
/profil

mengarah ke aplikasi PHP tanpa menampilkan:
/profil.php

Verifikasi
curl http://<hostname>/
curl http://<hostname>/profil

10. Reverse Proxy
Soal 11 — Reverse Proxy
Penny
Client
  |
  v
Penny
Apache Reverse Proxy
  |
  +----> Obladi
  |
  +----> Desmond

Abbey
Client
  |
  v
Abbey
Nginx Reverse Proxy
  |
  +----> Oblada
  |
  +----> Molly

Header
Forward:
Host
X-Real-IP

Tujuan
Backend harus dapat mengetahui:
hostname request

dan:
IP client asli

Verifikasi
curl -v http://www.<xxxx>.com
curl -v http://static.<xxxx>.com

Lakukan beberapa request dan amati backend yang menerima request.
Soal 12 — Basic Authentication
Target
penny

Path:
/admin

Credential
Username:
prabs

Password:
<ISI_PASSWORD_SESUAI_SOAL>

Tanpa Credential
curl -i http://penny.<xxxx>.com/admin

Expected:
401 Unauthorized

Dengan Credential
curl -u prabs:<PASSWORD> -i \
http://penny.<xxxx>.com/admin

Expected:
200 OK

Soal 13 — HTTP Redirect
Penny
IP/domain Penny:
301
 ↓
www.<xxxx>.com

Abbey
IP/domain Abbey:
302
 ↓
static.<xxxx>.com

Verifikasi
curl -I http://<IP-PENNY>

Expected:
HTTP/1.1 301
Location: http://www.<xxxx>.com

Kemudian:
curl -I http://<IP-ABBEY>

Expected:
HTTP/1.1 302
Location: http://static.<xxxx>.com

Soal 14 — Client IP Logging
Tujuan
Backend harus mencatat IP client asli.
Alur
Client
  |
  | IP asli
  v
Proxy
  |
  | X-Real-IP
  v
Backend
  |
  v
Access Log

Verifikasi
Lakukan request dari client:
curl http://www.<xxxx>.com

Kemudian cek log backend:
tail -f <ACCESS-LOG>

Pastikan IP yang muncul adalah:
IP CLIENT

bukan:
IP PENNY

atau:
IP ABBEY

Soal 15 — Path Proxy
Penny
Path:
/eternal

Directory:
/var/www/eternal

PHP:
ENABLED

Abbey
Path:
/orion

Directory:
/var/www/orion

PHP:
DISABLED

Verifikasi
curl http://penny.<xxxx>.com/eternal

dan:
curl http://abbey.<xxxx>.com/orion

11. Testing
Soal 16 — ApacheBench
Parameter
Requests      : 250
Concurrency   : 10

Target 1
www.<xxxx>.com

Contoh:
ab -n 250 -c 10 http://www.<xxxx>.com/

Target 2
static.<xxxx>.com

Contoh:
ab -n 250 -c 10 http://static.<xxxx>.com/

Hasil
www
Complete requests:
Failed requests:
Requests per second:
Time per request:
Transfer rate:

static
Complete requests:
Failed requests:
Requests per second:
Time per request:
Transfer rate:

Dokumentasi
  
Soal 17 — TXT Record
Target
alpha
beta
gamma
delta
epsilon

Contoh
alpha.<xxxx>.com TXT "alpha"

Verifikasi
dig TXT alpha.<xxxx>.com
dig TXT beta.<xxxx>.com
dig TXT gamma.<xxxx>.com
dig TXT delta.<xxxx>.com
dig TXT epsilon.<xxxx>.com

Soal 18 — DNS TTL
Target
abbey.<xxxx>.com

TTL
15 detik

Kondisi
Fase 1 — Sebelum perubahan
abbey
 ↓
IP LAMA

Fase 2 — Setelah perubahan tetapi TTL belum habis
abbey
 ↓
CACHE
 ↓
IP LAMA

Fase 3 — Setelah TTL habis
abbey
 ↓
QUERY BARU
 ↓
IP FIKTIF BARU

Serial
Setelah perubahan:
SOA SERIAL

harus dinaikkan.
Kemudian pastikan:
prab serial == tedd serial

Catatan
Setelah praktikum nomor 18 selesai, konfigurasi dikembalikan normal untuk kebutuhan nomor 20.

Soal 19 — CNAME External
Record
outbound.<xxxx>.com
        |
        v
http.badssl.com

Verifikasi DNS
dig CNAME outbound.<xxxx>.com

Verifikasi HTTP
curl http://outbound.<xxxx>.com

Output harus sesuai dengan konten tujuan yang ditentukan pada soal.
12. Persistence
Soal 20 — Autostart dan Persistence
Tujuan
Memastikan seluruh konfigurasi dan service tetap berjalan setelah restart.
Sebelum Restart
Tes:
ip addr
ip route

DNS:
dig <xxxx>.com

Web:
curl http://www.<xxxx>.com

Proxy:
curl http://static.<xxxx>.com

Restart
reboot

Setelah Restart
Periksa:
ip addr
ip route

Periksa service:
rc-status

atau command service yang sesuai dengan sistem.
Kemudian ulangi pengujian:
dig <xxxx>.com
curl http://www.<xxxx>.com
curl http://static.<xxxx>.com

Checklist
- [ ] IP tetap ada
- [ ] Gateway tetap ada
- [ ] Routing tetap ada
- [ ] DNS tetap aktif
- [ ] Apache tetap aktif
- [ ] Nginx tetap aktif
- [ ] PHP-FPM tetap aktif
- [ ] Reverse proxy tetap bekerja
- [ ] Konfigurasi tetap tersimpan
13. Alur Keseluruhan
Berikut alur keseluruhan praktikum:
                    INTERNET
                        |
                       NAT
                        |
                    rootkit
                        |
              INTERNAL ROUTING
                        |
        +---------------+---------------+
        |                               |
       DNS                             CLIENT
        |                               |
   +----+----+                          |
   |         |                          |
  prab      tedd                        |
 MASTER     SLAVE                        |
   |         ^                           |
   |         |                           |
   +----ZONE-TRANSFER-------------------+
                        |
                        v
                   DNS RESOLUTION
                        |
                        v
                www.<xxxx>.com
                        |
                      CNAME
                        |
                        v
                      Penny
                        |
                 Reverse Proxy
                        |
                +-------+-------+
                |               |
              Obladi         Desmond
               STATIC          STATIC

Untuk dynamic:
Client
  |
  v
DNS
  |
  v
static/core hostname
  |
  v
Abbey
  |
  v
Reverse Proxy
  |
  +---------> Oblada
  |
  +---------> Molly
               |
               v
            PHP-FPM

14. Konsep Penting yang Harus Dipahami
IP Address
Host → IP

Routing
Menentukan jalur packet

NAT
Private IP → alamat WAN

DNS
Hostname → IP

Reverse DNS
IP → Hostname

A Record
Hostname → IPv4

CNAME
Hostname → Hostname

PTR
IP → Hostname

TXT
Hostname → Text

SOA
Informasi authority dan versi zone

Zone Transfer
Master → Slave

Reverse Proxy
Client → Proxy → Backend

PHP-FPM
Nginx → PHP-FPM → PHP

Basic Authentication
Request
 ↓
Credential
 ↓
Access / 401

HTTP Redirect
301 = permanent
302 = temporary

TTL
Berapa lama DNS response dapat disimpan dalam cache

15. Persiapan Demo
Hal yang Harus Bisa Dijelaskan
Networking
- [ ] Fungsi rootkit
- [ ] Fungsi gateway
- [ ] Fungsi routing
- [ ] Fungsi NAT
- [ ] Fungsi resolver
DNS
- [ ] A record
- [ ] NS record
- [ ] SOA
- [ ] CNAME
- [ ] PTR
- [ ] TXT
- [ ] Master dan slave
- [ ] Zone transfer
- [ ] Serial
- [ ] TTL
- [ ] Forwarder
Web
- [ ] Apache
- [ ] Nginx
- [ ] PHP-FPM
- [ ] FastCGI
- [ ] Autoindex
- [ ] Rewrite
Proxy
- [ ] Reverse proxy
- [ ] Backend
- [ ] Load balancing
- [ ] Host header
- [ ] X-Real-IP
- [ ] Access log
HTTP
- [ ] 200
- [ ] 301
- [ ] 302
- [ ] 401
Testing
- [ ] curl
- [ ] dig
- [ ] nslookup
- [ ] ApacheBench
16. Pertanyaan yang Mungkin Ditanyakan Asisten
Networking
Q: Apa fungsi rootkit?
A:
rootkit merupakan router sentral dan gateway
yang menghubungkan jaringan-jaringan internal.

Q: Apa fungsi NAT?
A:
NAT melakukan translasi alamat sehingga host
dengan private IP dapat berkomunikasi melalui WAN.

Q: Apa fungsi default gateway?
A:
Default gateway merupakan next-hop yang digunakan
host untuk mencapai jaringan di luar subnet lokal.

DNS
Q: Apa perbedaan A dan CNAME?
A:
A     → hostname ke IPv4
CNAME → hostname ke hostname

Q: Apa fungsi SOA serial?
A:
Serial digunakan sebagai penanda versi zone.
Slave dapat mengetahui apakah zone master lebih baru.

Q: Kenapa ada prab dan tedd?
A:
prab berfungsi sebagai master authoritative,
sedangkan tedd sebagai slave/secondary.

Q: Apa itu zone transfer?
A:
Proses pemindahan/sinkronisasi informasi zone
dari master ke slave.

Q: Apa perbedaan forward dan reverse DNS?
A:
Forward:
hostname → IP

Reverse:
IP → hostname

Web Server
Q: Apa perbedaan Apache dan Nginx pada praktikum?
A:
Apache digunakan untuk web static dan reverse proxy Penny.

Nginx digunakan untuk web dynamic dan reverse proxy Abbey.

Q: Apa fungsi PHP-FPM?
A:
PHP-FPM menjalankan aplikasi PHP yang diteruskan
oleh Nginx melalui FastCGI.

Reverse Proxy
Q: Apa itu reverse proxy?
A:
Reverse proxy merupakan server perantara yang menerima
request client kemudian meneruskannya ke backend.

Q: Kenapa menggunakan X-Real-IP?
A:
Untuk meneruskan informasi IP client asli ke backend,
karena koneksi langsung backend berasal dari proxy.

HTTP
Q: Apa perbedaan 301 dan 302?
A:
301 = permanent redirect
302 = temporary redirect

Q: Apa arti 401?
A:
401 Unauthorized berarti request membutuhkan
authentication yang valid.

DNS Cache
Q: Kenapa perubahan DNS tidak langsung terlihat?
A:
Karena resolver/client dapat masih memiliki
jawaban lama di cache sampai TTL habis.

17. Final Checklist
Network
- [ ] Semua node hidup
- [ ] IP address benar
- [ ] Gateway benar
- [ ] Routing benar
- [ ] NAT aktif
- [ ] Internet dapat diakses
DNS
- [ ] prab aktif
- [ ] tedd aktif
- [ ] SOA benar
- [ ] NS benar
- [ ] A record benar
- [ ] CNAME benar
- [ ] PTR benar
- [ ] TXT benar
- [ ] Zone transfer berhasil
- [ ] Serial sama
- [ ] TTL nomor 18 berhasil diuji
Web
- [ ] Obladi aktif
- [ ] Desmond aktif
- [ ] Oblada aktif
- [ ] Molly aktif
- [ ] Apache aktif
- [ ] Nginx aktif
- [ ] PHP-FPM aktif
- [ ] /arsip/ bekerja
- [ ] /profil bekerja
Reverse Proxy
- [ ] Penny aktif
- [ ] Abbey aktif
- [ ] Vault bekerja
- [ ] Core bekerja
- [ ] Host header diteruskan
- [ ] X-Real-IP diteruskan
- [ ] Access log menunjukkan IP client
Security / HTTP
- [ ] /admin membutuhkan authentication
- [ ] Credential valid dapat masuk
- [ ] Penny menggunakan 301
- [ ] Abbey menggunakan 302
Testing
- [ ] ApacheBench 250 requests
- [ ] Concurrency 10
- [ ] www tested
- [ ] static tested
- [ ] TXT tested
- [ ] External CNAME tested
Persistence
- [ ] Reboot berhasil
- [ ] Network kembali aktif
- [ ] DNS kembali aktif
- [ ] Web server kembali aktif
- [ ] Reverse proxy kembali aktif
- [ ] Konfigurasi tetap tersimpan
18. Kesimpulan
Praktikum ini membangun jaringan secara bertahap mulai dari:
IP Address
    ↓
Routing
    ↓
NAT
    ↓
DNS
    ↓
Zone Transfer
    ↓
Web Server
    ↓
Reverse Proxy
    ↓
Authentication
    ↓
Redirect
    ↓
Logging
    ↓
Benchmark
    ↓
DNS Cache
    ↓
Persistence

Setiap bagian saling berhubungan.
DNS menyediakan resolusi nama yang digunakan untuk mengakses web server. Reverse proxy menjadi gerbang menuju backend. Header Host dan X-Real-IP memungkinkan backend mengetahui konteks request dan alamat client. DNS TTL kemudian digunakan untuk mengamati perilaku caching. Pada tahap akhir, seluruh konfigurasi diuji kembali setelah restart untuk memastikan persistence.
19. Dokumentasi
Screenshots
Seluruh bukti praktikum disimpan di:
screenshots/

Script
Seluruh script disimpan di:
scripts/

Configuration
Seluruh konfigurasi disimpan di:
configs/

20. Author
Kelompok : K-XX
Praktikan:
1. Nama 1
2. Nama 2
3. Nama 3
4. Nama 4

Praktikum:
Komunikasi Data & Jaringan Komputer 2026

Modul:
Modul 2 — Shadow Net Operation

End
Dokumentasi ini dibuat sebagai dokumentasi konfigurasi, pengujian, troubleshooting, dan bahan persiapan demo praktikum.


Template di atas sengaja dibuat sebagai **README dokumentasi sekaligus bahan belajar demo**, bukan cuma laporan ha
