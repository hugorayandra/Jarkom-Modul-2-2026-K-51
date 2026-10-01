Jadi README praktikum kamu jangan dibuat seperti tutorial generik, tetapi seperti laporan pengerjaan praktikum. Contohnya pola yang dipakai repository Jarkom lain juga adalah ## Nomor 1, ## Nomor 2, kemudian ## Jawaban, konfigurasi, testing, dan seterusnya. GitHub
Untuk kasusmu, struktur yang tepat adalah seperti ini:
# Laporan Praktikum Jaringan Komputer 2026

## Kelompok K-51

| Nama | NRP |
|---|---|
| Muhammad Hugo Rayandra E | 5027251076 |
| Arrumanta Ekna Luhkinasih | 5027251044 |

---

# Daftar Isi

- [Nomor 1 - Topologi dan Konfigurasi Awal](#nomor-1---topologi-dan-konfigurasi-awal)
- [Nomor 2 - Konfigurasi Router](#nomor-2---konfigurasi-router)
- [Nomor 3 - Konfigurasi Client](#nomor-3---konfigurasi-client)
- [Nomor 4 - Pengujian Koneksi](#nomor-4---pengujian-koneksi)
- [Nomor 5 - Persistence dan Script Verifikasi](#nomor-5---persistence-dan-script-verifikasi)
- [Nomor 6 - DNS dan ICMP](#nomor-6---dns-dan-icmp)
- [Nomor 7 - FTP Setup](#nomor-7---ftp-setup)
- [Nomor 8 - FTP Knights](#nomor-8---ftp-knights)
- [Nomor 9 - FTP Mika](#nomor-9---ftp-mika)
- [Nomor 10 - ICMP Traffic](#nomor-10---icmp-traffic)
- [Nomor 11 - Telnet](#nomor-11---telnet)
- [Nomor 12 - Nmap](#nomor-12---nmap)
- [Nomor 13 - SSH](#nomor-13---ssh)
- [Nomor 14 - HTTP Brute Force](#nomor-14---http-brute-force)
- [Nomor 15 - USB HID](#nomor-15---usb-hid)
- [Nomor 16 - FTP Malware](#nomor-16---ftp-malware)
- [Nomor 17 - Malware Download](#nomor-17---malware-download)
- [Nomor 18 - SMB Transfer](#nomor-18---smb-transfer)
- [Nomor 19 - SMTP Threat](#nomor-19---smtp-threat)
- [Nomor 20 - TLS](#nomor-20---tls)
- [Dokumentasi](#dokumentasi)
- [Kendala](#kendala)
- [Kesimpulan](#kesimpulan)

---

# Nomor 1 - Topologi dan Konfigurasi Awal

> Buatlah topologi jaringan sesuai dengan pembagian yang telah
> ditentukan pada modul praktikum.

## Topologi

Topologi yang digunakan pada kelompok K-51 terdiri dari Router Lain,
tiga switch, dan lima client.

```text
                         NAT1
                           |
                         eth0
                           |
                    +-------------+
                    |    Lain     |
                    |   Router    |
                    +-------------+
                     |     |     |
                   eth1   eth2   eth3
                     |     |     |
                  Switch1 Switch2 Switch3
                  /    \     |     /    \
              Alice   Mika  Chisa Knights Eiri

Pembagian Network
Network	Gateway	Client
10.89.1.0/24	10.89.1.1	Alice, Mika
10.89.2.0/24	10.89.2.1	Chisa
10.89.3.0/24	10.89.3.1	Knights, Eiri

```


# Nomor 2 - Konfigurasi Router

Nomor 2 - Konfigurasi Router
Konfigurasikan Router Lain sebagai router yang menghubungkan
seluruh subnet dan jaringan luar.

Konfigurasi Interface
Pada Router Lain jalankan:
```bash
ip link set eth0 up
ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
```
Kemudian cek interface:
```bash
ip -br a
```

Konfigurasi IP
```bash
ip addr add 10.89.1.1/24 dev eth1
ip addr add 10.89.2.1/24 dev eth2
ip addr add 10.89.3.1/24 dev eth3
```
Kemudian cek:
```bash
ip addr
ip route
```

Mengaktifkan IPv4 Forwarding
```bash
sysctl -w net.ipv4.ip_forward=1
```
Untuk memastikan forwarding aktif:
```bash
cat /proc/sys/net/ipv4/ip_forward
```
Jika output:
1

maka IPv4 forwarding telah aktif.
Konfigurasi NAT
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Kemudian lakukan pengecekan:
```bash
iptables -t nat -L -v -n
```
