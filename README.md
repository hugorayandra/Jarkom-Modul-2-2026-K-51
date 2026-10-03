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

![Nama Gambar](path/ke/gambar.png)

---

