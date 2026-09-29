# Jarkom-Modul-2-2026-K-51

### Shadow Net Operation — The Mesh

---

## Identitas Kelompok K-51

| No. | Nama                         |     NRP    |
| :-: | ---------------------------- | :--------: |
|  1  | Arrumanta Ekna Luhkinasih | 5027251044 |
|  2  | Muhammad Hugo Rayandra E   | 5027251076 |

---

# 1. Pendahuluan

## 1.1 Latar Belakang

Praktikum Modul 2 membahas pembangunan sebuah jaringan terisolasi bernama **The Mesh**. Jaringan terdiri atas beberapa entitas yang memiliki fungsi berbeda, mulai dari router, client, DNS server, reverse proxy, web server statis, hingga web server dinamis.

Praktikum dilakukan menggunakan GNS3 dengan beberapa node berbasis Linux. Setiap node diberikan alamat IP dan hostname sesuai dengan fungsi yang telah ditentukan. Setelah jaringan terbentuk, dilakukan konfigurasi routing, NAT, DNS authoritative dan slave, web server, reverse proxy, authentication, logging, benchmark, hingga persistence service.

Konfigurasi DNS menjadi salah satu fondasi utama karena sebagian besar layanan berikutnya diakses menggunakan hostname, bukan secara langsung menggunakan alamat IP.

## 1.2 Tujuan

Praktikum ini bertujuan untuk:

1. Membangun topologi jaringan The Mesh.
2. Mengonfigurasi alamat IP dan routing antar jaringan.
3. Mengaktifkan NAT pada router utama.
4. Membangun DNS authoritative pada prab dan DNS slave pada tedd.
5. Melakukan zone transfer.
6. Membuat hostname dan record DNS untuk seluruh entitas.
7. Mengonfigurasi web server statis dan dinamis.
8. Mengimplementasikan reverse proxy menggunakan Apache dan Nginx.
9. Menerapkan authentication dan redirect.
10. Memastikan client IP dapat diteruskan dan tercatat pada access log.
11. Melakukan stress test menggunakan ApacheBench.
12. Menguji TXT record, TTL/cache DNS, CNAME, dan persistence service.

---

# 2. Topologi dan Pembagian Entitas

The Mesh terdiri dari beberapa kelompok entitas dengan fungsi sebagai berikut.

| Entitas | Fungsi                         |
| ------- | ------------------------------ |
| rootkit | Router/Gateway utama           |
| alpha   | Client sayap kiri              |
| beta    | Client sayap kiri              |
| gamma   | Client sayap kiri              |
| delta   | Client sayap kanan             |
| epsilon | Client sayap kanan             |
| prab    | DNS Master / NS1               |
| tedd    | DNS Slave / NS2                |
| abbey   | Reverse Proxy untuk area core  |
| penny   | Reverse Proxy untuk area vault |
| obladi  | Web statis                     |
| desmond | Web statis                     |
| oblada  | Web dinamis                    |
| molly   | Web dinamis                    |

Domain yang digunakan dalam praktikum adalah:

**K-51.com**

---

# 3. Konfigurasi IP Address

Pembagian alamat IP yang digunakan pada The Mesh adalah sebagai berikut.

| Host    | IP Address |
| ------- | ---------- |
| rootkit | 10.89.5.1  |
| alpha   | 10.89.1.10 |
| beta    | 10.89.1.11 |
| gamma   | 10.89.1.12 |
| delta   | 10.89.2.10 |
| epsilon | 10.89.2.11 |
| abbey   | 10.89.3.10 |
| penny   | 10.89.4.10 |
| prab    | 10.89.5.10 |
| tedd    | 10.89.5.11 |
| obladi  | 10.89.5.12 |
| desmond | 10.89.5.13 |
| oblada  | 10.89.5.14 |
| molly   | 10.89.5.15 |

Konfigurasi dilakukan pada masing-masing node sesuai dengan subnet yang telah ditentukan.

**Bukti:**

> [Screenshot konfigurasi IP seluruh node]

---

# 4. Konfigurasi DNS Authoritative dan Slave

Pada node **prab** dibangun DNS authoritative untuk domain **K-51.com**. DNS prab memiliki SOA yang menunjuk ke `prab.K-51.com` serta NS record untuk `prab.K-51.com` dan `tedd.K-51.com`.

Record utama yang digunakan antara lain:

```text
K-51.com.       IN A     10.89.4.10
prab.K-51.com.  IN A     10.89.5.10
tedd.K-51.com.  IN A     10.89.5.11
```

DNS pada prab juga dikonfigurasi agar melakukan notify dan mengizinkan zone transfer kepada tedd.

Pada node **tedd**, zona K-51.com dikonfigurasi sebagai slave dengan master:

```text
masters { 10.89.5.10; };
```

Hasil pengujian DNS menunjukkan bahwa domain apex berhasil dijawab secara authoritative.

Contoh:

```text
dig @10.89.5.10 K-51.com
```

Hasil:

```text
status: NOERROR
ANSWER: 1

K-51.com.    86400    IN    A    10.89.4.10
```

Pada tedd:

```text
dig @10.89.5.11 K-51.com
```

Hasil juga menunjukkan:

```text
status: NOERROR
flags: qr aa rd ra
```

Hal tersebut menunjukkan bahwa tedd telah dapat menjawab query secara authoritative.

**Bukti:**

> [Screenshot konfigurasi prab]
> [Screenshot konfigurasi tedd]
> [Screenshot hasil dig dari prab]
> [Screenshot hasil dig dari tedd]

---

# 5. Konfigurasi Hostname Seluruh Entitas

Seluruh node diberikan hostname sesuai dengan glosarium The Mesh.

Hostname yang digunakan adalah:

```text
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
```

Hostname diverifikasi menggunakan perintah:

```bash
hostname
```

dan:

```bash
hostnamectl
```

Setiap node juga memiliki record DNS sesuai hostname masing-masing.

**Bukti:**

> [Screenshot hostname beberapa node]
> [Screenshot DNS record]

---

# 6. Verifikasi Zone Transfer

Zone transfer antara **prab sebagai master** dan **tedd sebagai slave** berhasil dilakukan.

Pada prab digunakan konfigurasi:

```text
allow-transfer {
    10.89.5.11;
};

also-notify {
    10.89.5.11;
};
```

Pada tedd digunakan konfigurasi slave:

```text
zone "K-51.com" {
    type slave;
    file "/var/cache/bind/db.K-51.com";
    masters { 10.89.5.10; };
};
```

Serial SOA pada master dan slave digunakan untuk memastikan bahwa kedua server memiliki versi zona yang sama.

Verifikasi dilakukan menggunakan:

```bash
dig @10.89.5.10 K-51.com SOA +short
```

dan:

```bash
dig @10.89.5.11 K-51.com SOA +short
```

Hasil menunjukkan serial SOA yang sama sehingga zona pada tedd telah tersinkronisasi dengan prab.

**Bukti:**

> [Screenshot SOA prab]
> [Screenshot SOA tedd]

---

# 7. Konfigurasi Record Vault, Core, dan CNAME

Pada zona **K-51.com** ditambahkan record untuk area vault dan core.

Area vault terdiri dari:

```text
obladi
desmond
```

Sedangkan area core terdiri dari:

```text
oblada
molly
```

Record DNS yang digunakan:

```text
vault.K-51.com.    IN A    10.89.5.12
vault.K-51.com.    IN A    10.89.5.13

core.K-51.com.     IN A    10.89.5.14
core.K-51.com.     IN A    10.89.5.15
```

Kemudian dibuat CNAME:

```text
www.K-51.com.      IN CNAME    penny.K-51.com.
static.K-51.com.   IN CNAME    abbey.K-51.com.
```

Pengujian dilakukan dari client menggunakan:

```bash
dig @10.89.5.10 vault.K-51.com
dig @10.89.5.10 core.K-51.com
dig @10.89.5.10 www.K-51.com
dig @10.89.5.10 static.K-51.com
```

Hasil query digunakan untuk memastikan hostname mengarah ke tujuan yang telah ditentukan.

**Bukti:**

> [Screenshot record vault]
> [Screenshot record core]
> [Screenshot CNAME]
> [Screenshot hasil dig dari client]

---

# 8. Reverse DNS

Reverse DNS dikonfigurasi pada prab untuk segmen jaringan tempat gateway dan repository berada.

Reverse zone digunakan untuk melakukan pemetaan:

```text
IP Address → Hostname
```

PTR record dibuat untuk hostname:

```text
abbey
penny
obladi
desmond
oblada
molly
```

Zona reverse kemudian ditarik oleh tedd sebagai slave.

Pengujian dilakukan menggunakan:

```bash
dig -x 10.89.3.10
dig -x 10.89.4.10
dig -x 10.89.5.12
dig -x 10.89.5.13
```

Hasil query diharapkan mengembalikan hostname yang sesuai.

**Bukti:**

> [Screenshot konfigurasi reverse zone prab]
> [Screenshot reverse zone tedd]
> [Screenshot dig -x]

---

# 9. Web Server Statis

Node **obladi** dan **desmond** digunakan sebagai web server statis menggunakan Apache.

Directory:

```text
/arsip/
```

dikonfigurasi agar dapat menampilkan directory listing menggunakan fitur autoindex Apache.

Pengujian dilakukan menggunakan hostname:

```text
http://vault.K-51.com/arsip/
```

Pengujian tidak dilakukan menggunakan IP secara langsung karena soal mewajibkan akses melalui hostname.

**Bukti:**

> [Screenshot konfigurasi Apache]
> [Screenshot directory /arsip/]
> [Screenshot browser]

---

# 10. Web Server Dinamis

Node **oblada** dan **molly** digunakan sebagai web server dinamis menggunakan Nginx dan PHP-FPM.

Aplikasi menyediakan halaman:

```text
/
```

dan:

```text
/profil
```

URL `/profil` dikonfigurasi menggunakan rewrite sehingga tidak perlu menuliskan `.php`.

Contoh akses:

```text
http://core.K-51.com/
http://core.K-51.com/profil
```

Pengujian dilakukan menggunakan hostname.

**Bukti:**

> [Screenshot Nginx]
> [Screenshot PHP-FPM]
> [Screenshot halaman utama]
> [Screenshot halaman /profil]

---

# 11. Reverse Proxy Penny dan Abbey

Node **penny** digunakan sebagai reverse proxy Apache menuju area vault:

```text
penny → obladi
      → desmond
```

Sedangkan **abbey** menggunakan Nginx sebagai reverse proxy menuju area core:

```text
abbey → oblada
      → molly
```

Header berikut diteruskan oleh reverse proxy:

```text
Host
X-Real-IP
```

Konfigurasi ini memungkinkan backend mengetahui hostname yang digunakan client serta alamat IP asli client.

Pengujian dilakukan dengan mengakses:

```text
www.K-51.com
static.K-51.com
```

dan mengamati server backend yang menerima request.

**Bukti:**

> [Screenshot konfigurasi Penny]
> [Screenshot konfigurasi Abbey]
> [Screenshot backend Obladi/Desmond]
> [Screenshot backend Oblada/Molly]

---

# 12. Basic Authentication

Pada node **penny**, path:

```text
/admin
```

diberikan perlindungan menggunakan Basic Authentication.

Akses tanpa credential harus ditolak dan browser menampilkan permintaan username dan password.

Pengujian dilakukan dengan credential yang telah ditentukan pada soal praktikum.

Hasil pengujian menunjukkan bahwa endpoint `/admin` hanya dapat diakses setelah autentikasi berhasil.

**Bukti:**

> [Screenshot akses tanpa credential]
> [Screenshot akses setelah authentication berhasil]

---

# 13. Redirect Canonical Hostname

Konfigurasi redirect dibuat agar setiap akses menuju gateway menggunakan hostname kanonik.

Untuk Penny digunakan redirect permanen:

```text
301 → www.K-51.com
```

Sedangkan Abbey menggunakan redirect sementara:

```text
302 → static.K-51.com
```

Pengujian dilakukan menggunakan:

```bash
curl -I http://10.89.4.10
curl -I http://penny.K-51.com
curl -I http://10.89.3.10
curl -I http://abbey.K-51.com
```

Header `Location` digunakan untuk memastikan tujuan redirect sesuai konfigurasi.

**Bukti:**

> [Screenshot curl status 301]
> [Screenshot curl status 302]

---

# 14. Access Log dan Client IP

Reverse proxy dikonfigurasi agar meneruskan IP asli client melalui:

```text
X-Real-IP
```

Backend kemudian dikonfigurasi agar access log mencatat IP client asli, bukan alamat IP Penny atau Abbey.

Pengujian dilakukan dengan mengakses layanan melalui reverse proxy kemudian memeriksa access log pada backend.

Contoh:

```bash
tail -f /var/log/nginx/access.log
```

atau:

```bash
tail -f /var/log/apache2/access.log
```

Hasil pengujian menunjukkan alamat IP client dapat diteruskan sampai ke backend.

**Bukti:**

> [Screenshot request dari client]
> [Screenshot access log backend]

---

# 15. Proxy Path Eternal dan Orion

Pada **penny** dibuat reverse proxy khusus untuk:

```text
/eternal
```

yang mengarah ke:

```text
/var/www/eternal
```

Path tersebut mendukung rendering PHP.

Pada **abbey** dibuat path:

```text
/orion
```

yang mengarah ke:

```text
/var/www/orion
```

Path `/orion` digunakan sebagai layanan statis dan tidak melakukan rendering PHP.

Pengujian dilakukan melalui hostname dan path masing-masing.

**Bukti:**

> [Screenshot konfigurasi Penny /eternal]
> [Screenshot konfigurasi Abbey /orion]
> [Screenshot hasil akses browser]

---

# 16. Stress Test ApacheBench

Pengujian benchmark dilakukan menggunakan ApacheBench dari salah satu client, misalnya Alpha.

Parameter yang digunakan:

```text
Requests     : 250
Concurrency  : 10
```

Pengujian dilakukan pada:

```text
www.K-51.com
static.K-51.com
```

Contoh perintah:

```bash
ab -n 250 -c 10 http://www.K-51.com/
```

dan:

```bash
ab -n 250 -c 10 http://static.K-51.com/
```

Hasil benchmark dicatat untuk membandingkan performa kedua endpoint.

Parameter yang diamati antara lain:

* Complete requests
* Failed requests
* Requests per second
* Time per request
* Transfer rate

**Hasil:**

### [www.K-51.com](http://www.K-51.com)

> [Masukkan output ApacheBench]

### static.K-51.com

> [Masukkan output ApacheBench]

---

# 17. TXT Record

TXT record ditambahkan untuk seluruh client sayap kiri dan kanan:

```text
alpha
beta
gamma
delta
epsilon
```

Contoh:

```text
alpha.K-51.com.    IN TXT    "alpha"
beta.K-51.com.     IN TXT    "beta"
gamma.K-51.com.    IN TXT    "gamma"
delta.K-51.com.    IN TXT    "delta"
epsilon.K-51.com.  IN TXT    "epsilon"
```

Verifikasi dilakukan dengan:

```bash
dig @10.89.5.10 alpha.K-51.com TXT
dig @10.89.5.10 beta.K-51.com TXT
dig @10.89.5.10 gamma.K-51.com TXT
dig @10.89.5.10 delta.K-51.com TXT
dig @10.89.5.10 epsilon.K-51.com TXT
```

Hasil query harus mengembalikan teks hostname masing-masing.

**Bukti:**

> [Screenshot hasil TXT record]

---

# 18. Pengujian TTL dan DNS Cache

A record `abbey.K-51.com` diubah menjadi alamat IP fiktif yang tetap memiliki format IPv4 valid.

Setelah perubahan, serial SOA pada prab dinaikkan sehingga perubahan dapat ditransfer ke tedd.

TTL record yang diuji ditetapkan sebesar:

```text
15 detik
```

Pengujian dilakukan dalam tiga kondisi:

### Fase 1 — Sebelum perubahan

DNS masih mengembalikan IP lama.

### Fase 2 — Setelah perubahan tetapi TTL belum habis

Client masih mendapatkan IP lama karena record masih berada di cache.

### Fase 3 — Setelah TTL habis

DNS mengembalikan IP baru setelah cache expired.

Hasil pengujian menunjukkan pengaruh TTL terhadap proses caching DNS.

**Bukti:**

> [Screenshot sebelum perubahan]
> [Screenshot setelah perubahan < 15 detik]
> [Screenshot setelah TTL habis]

---

# 19. CNAME Outbound

Dibuat CNAME:

```text
outbound.K-51.com
```

yang mengarah ke:

```text
http.badssl.com
```

Pengujian dilakukan menggunakan:

```bash
curl http://outbound.K-51.com
```

Hasil output kemudian dibandingkan dengan konten dari endpoint tujuan.

Pengujian ini digunakan untuk memastikan resolusi CNAME dan akses menuju tujuan eksternal dapat berjalan.

**Bukti:**

> [Screenshot DNS CNAME]
> [Screenshot hasil curl]

---

# 20. Persistence dan Autostart

Setelah seluruh konfigurasi selesai, setiap service diperiksa agar tetap berjalan setelah node melakukan restart.

Service yang diperiksa meliputi:

* BIND
* Apache
* Nginx
* PHP-FPM
* service jaringan
* konfigurasi reverse proxy
* konfigurasi DNS

Konfigurasi nomor 18 dikembalikan ke kondisi normal sesuai instruksi praktikum.

Setelah restart, setiap service diverifikasi kembali menggunakan command yang sesuai.

Contoh:

```bash
systemctl status <service>
```

atau command pemeriksaan service yang tersedia pada image yang digunakan.

Selain itu dilakukan pengujian ulang terhadap DNS dan web service untuk memastikan konfigurasi tetap berjalan setelah restart.

**Bukti:**

> [Screenshot service sebelum restart]
> [Screenshot node setelah restart]
> [Screenshot service setelah restart]
> [Screenshot pengujian DNS setelah restart]
> [Screenshot pengujian web setelah restart]

---

# 21. Hasil dan Pembahasan

Berdasarkan konfigurasi yang dilakukan, The Mesh dibangun menggunakan beberapa komponen yang saling terhubung. Rootkit berfungsi sebagai router utama, sedangkan prab dan tedd berfungsi sebagai DNS server master dan slave.

DNS menjadi dasar bagi layanan lainnya karena hostname seperti `www.K-51.com`, `static.K-51.com`, `vault.K-51.com`, dan `core.K-51.com` digunakan untuk mengakses layanan.

Zone transfer memungkinkan tedd memiliki salinan zona dari prab sehingga DNS tetap dapat memberikan layanan apabila query diarahkan ke server kedua.

Setelah DNS berjalan, layanan web dapat dibangun di area vault dan core. Reverse proxy pada Penny dan Abbey kemudian menjadi gerbang bagi client sebelum request diteruskan menuju backend.

Selain layanan utama, praktikum juga menguji fitur DNS yang lebih lanjut seperti reverse DNS, TXT record, CNAME, TTL dan caching. Pada sisi web dilakukan pengujian authentication, redirect, forwarding client IP, path proxy, serta ApacheBench.

Dengan demikian, praktikum menggabungkan beberapa konsep jaringan menjadi satu infrastruktur yang saling terintegrasi.

---

# 22. Kesimpulan

Praktikum Modul 2 memberikan implementasi langsung terhadap konsep komunikasi data dan jaringan komputer melalui pembangunan jaringan The Mesh. Konfigurasi dilakukan mulai dari layer jaringan, routing, NAT, DNS, hingga layanan aplikasi berbasis web.

DNS authoritative dan slave digunakan untuk menyediakan resolusi nama internal, sedangkan zone transfer memastikan sinkronisasi antara prab dan tedd. Selanjutnya, web server statis dan dinamis ditempatkan pada area vault dan core dan diakses melalui reverse proxy.

Pengujian tambahan berupa reverse DNS, Basic Authentication, redirect, forwarding client IP, ApacheBench, TXT record, TTL caching, dan CNAME digunakan untuk memastikan berbagai fungsi jaringan dan aplikasi dapat bekerja sesuai rancangan.

Seluruh konfigurasi kemudian perlu dipastikan tetap berjalan setelah restart agar infrastruktur The Mesh dapat beroperasi secara konsisten.

---

# 23. Dokumentasi

Seluruh hasil konfigurasi dan pengujian dilengkapi dengan screenshot sebagai bukti pengerjaan.

Dokumentasi mencakup:

1. Konfigurasi IP.
2. Konfigurasi routing dan NAT.
3. Konfigurasi DNS master.
4. Konfigurasi DNS slave.
5. Hasil zone transfer.
6. Konfigurasi record DNS.
7. Hasil reverse DNS.
8. Konfigurasi Apache.
9. Konfigurasi Nginx.
10. Konfigurasi PHP-FPM.
11. Konfigurasi reverse proxy.
12. Basic Authentication.
13. Redirect.
14. Access log.
15. Proxy path.
16. ApacheBench.
17. TXT record.
18. TTL dan DNS cache.
19. CNAME dan curl.
20. Persistence service.

---

# 24. Daftar Bukti Pengujian

| No. | Pengujian         | Bukti                              |
| --- | ----------------- | ---------------------------------- |
| 1   | IP dan gateway    | Screenshot                         |
| 2   | NAT               | Screenshot                         |
| 3   | Routing           | Screenshot ping                    |
| 4   | DNS authoritative | Screenshot dig                     |
| 5   | Hostname          | Screenshot hostname                |
| 6   | Zone transfer     | Screenshot SOA prab/tedd           |
| 7   | Vault/Core/CNAME  | Screenshot dig                     |
| 8   | Reverse DNS       | Screenshot dig -x                  |
| 9   | Static web        | Screenshot browser                 |
| 10  | Dynamic web       | Screenshot browser                 |
| 11  | Reverse proxy     | Screenshot browser/log             |
| 12  | Basic auth        | Screenshot authentication          |
| 13  | Redirect          | Screenshot curl                    |
| 14  | Client IP log     | Screenshot access log              |
| 15  | Eternal/Orion     | Screenshot browser                 |
| 16  | ApacheBench       | Screenshot output                  |
| 17  | TXT record        | Screenshot dig TXT                 |
| 18  | TTL/cache         | Screenshot tiga fase               |
| 19  | CNAME/curl        | Screenshot curl                    |
| 20  | Persistence       | Screenshot service setelah restart |
