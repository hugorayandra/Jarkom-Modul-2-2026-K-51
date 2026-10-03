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
