# Jarkom_Modul5_Lapres_B09
## Pembuatan Topologi
Buat file `topologi.sh` sebagai berikut untuk menggambarkan topologi :  
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &
uml_switch -unix switch4 > /dev/null < /dev/null &
uml_switch -unix switch5 > /dev/null < /dev/null &
uml_switch -unix switch6 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.74.41 eth1=daemon,,,switch4 eth2=daemon,,,switch3 mem=96M &
xterm -T KEDIRI -e linux ubd0=KEDIRI,jarkom umid=KEDIRI eth0=daemon,,,switch4 eth1=daemon,,,switch1 eth2=daemon,,,switch6 mem=96M &
xterm -T BATU -e linux ubd0=BATU,jarkom umid=BATU eth0=daemon,,,switch3 eth1=daemon,,,switch5 eth2=daemon,,,switch2 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch1 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch1 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch5 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch6 mem=96M &
```
Dengan menggunakan teknik subnetting CIDR, didapat pembagian IP untuk setiap UML. Tambahkan interface berikut pada `/etc/network/interfaces/` untuk setiap UML :  
**SURABAYA**
```
auto eth0
iface eth0 inet static
address 10.151.74.42
netmask 255.255.255.252
gateway 10.151.74.41

auto eth1
iface eth1 inet static
address 192.168.2.1
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 192.168.5.1
netmask 255.255.255.252
```
**KEDIRI**
```
auto eth0
iface eth0 inet static
address 192.168.2.2
netmask 255.255.255.252
gateway 192.168.2.1

auto eth1
iface eth1 inet static
address 192.168.1.1
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.0.1
netmask 255.255.255.0
```
**BATU**
```
auto eth0
iface eth0 inet static
address 192.168.5.2
netmask 255.255.255.252
gateway 192.168.5.1

auto eth1
iface eth1 inet static
address 192.168.4.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 10.151.83.81
netmask 255.255.255.248
```
**GRESIK**
```
auto eth0
iface eth0 inet dhcp
```
**SIDOARJO**
```
auto eth0
iface eth0 inet dhcp
```
**MALANG**
```
auto eth0
iface eth0 inet static
address 10.151.83.82
netmask 255.255.255.248
gateway 10.151.83.81
```
**MOJOKERTO**
```
auto eth0
iface eth0 inet static
address 10.151.83.83
netmask 255.255.255.248
gateway 10.151.83.81
```
**MADIUN**
```
auto eth0
iface eth0 inet static
address 192.168.1.2
netmask 255.255.255.248
gateway 192.168.1.1
```
**PROBOLINGGO**
```
auto eth0
iface eth0 inet static
address 192.168.1.3
netmask 255.255.255.248
gateway 192.168.1.1
```
Tambahkan routing untuk UML SURABAYA  
**SURABAYA**
```
route add -net 192.168.0.0 netmask 255.255.254.0 gw 192.168.2.2
route add -net 192.168.4.0 netmask 255.255.255.0 gw 192.168.5.2
route add -net 10.151.83.80 netmask 255.255.255.248 gw 192.168.5.2
```
Setting UML MOJOKERTO agar menjadi DHCP Server dengan install dan ubah `dhcpd.conf` :  
```
subnet 10.151.83.80 netmask 255.255.255.248 {
}

subnet 192.168.4.0 netmask 255.255.255.0 {
    range 192.168.4.2 192.168.4.254;
    option routers 192.168.4.1;
    option broadcast-address 192.168.4.255;
    option domain-name-servers 10.151.83.82;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.2 192.168.0.254;
    option routers 192.168.0.1;
    option broadcast-address 192.168.0.255;
    option domain-name-servers 10.151.83.82;
    default-lease-time 600;
    max-lease-time 7200;
}
```
Lalu install DHCP Relay pada UML BATU dan KEDIRI dan masukan IP DHCP Server (UML MOJOKERTO) saat menginstall.
## IPTables
### Nomor 1
Mengkonfigurasi SURABAYA menggunakan iptables, namun tidak menggunakan MASQUERADE  
`iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 10.151.74.42`  
Penjelasan :
- Menggunakan POSTROUTING untuk mengubah source menjadi ip eht0 surabaya.
### Nomor 2
Mendrop semua akses SSH dari luar Topologi (UML) pada DHCP dan DNS Server pada SURABAYA  
`iptables -A FORWARD -p tcp --dport 22 -d 10.151.83.80/29 -i eth0 -j DROP`  
Penjelasan :
- Menggunakan FORWARD untuk drop semua koneksi yang destinasinya subnet IP DMZ dengan port 22.
### Nomor 3
Membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan yang berasal dari mana saja menggunakan iptables pada masing-masing server, selebihnya di DROP.  
`iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP`  
Penjelasan :
- Script iptables dijalankan di kedua server.
- Menggunakan INPUT untuk drop koneksi apabila koneksi diatas 3.
### Nomor 4 dan 5
Membatasi akses ke MALANG yang berasal dari subnet SIDOARJO dan GRESIK dengan ketentuan akses dari subnet SIDOARJO hanya diperbolehkan pada pukul 07.00 - 17.00 pada hari Senin
sampai Jumat dan akses dari subnet GRESIK hanya diperbolehkan pada pukul 17.00 hingga pukul 07.00 setiap harinya.  
```
iptables -A INPUT -s 192.168.4.0/24 -m time --timestart 07:00 --timestop 17:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 17:00 --timestop 00:00 -j ACCEPT
iptables -A INPUT -s 192.168.0.0/24 -m time --timestart 00:00 --timestop 07:00 -j ACCEPT
iptables -A INPUT -s 192.168.4.0/24 -j REJECT
iptables -A INPUT -s 192.168.0.0/24 -j REJECT
```  
Penjelasan :
- Menggunakan INPUT untuk menerima koneksi dari subnet SIDOARJO apabila sesuai ketentuan.
- Menggunakan INPUT untuk menerima koneksi dari subnet GRESIK apabila sesuai ketentuan.
- Selebihnya akan di REJECT.
### Nomor 6
SURABAYA disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada PROBOLINGGO port 80 dan MADIUN port 80  
```
iptables -A PREROUTING -t nat -p tcp -d 192.168.1.20 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.1.2
iptables -A PREROUTING -t nat -p tcp -d 192.168.1.20 --dport 80 -j DNAT --to-destination 192.168.1.3
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 192.168.1.2 -j SNAT --to-source 192.168.1.20
iptables -t nat -A POSTROUTING -p tcp --dport 80 -d 192.168.1.3 -j SNAT --to-source 192.168.1.20
```  
Penjelasan : 
- Mengunakan PREROUTING untuk mengubah tujuan paket dari yang asalnya ke DNS Server dengan port 80 jadi ke Web Server.
- Didistribusikan bergantian dengan menggunakan statistic opsi every diisi value 2.
- Menggunakan POSTROUTING untuk mengubah paket yang telah di preroute agar sumbernya terlihat berasal dari DNS Server.
### Nomor 7
Semua paket didrop oleh firewall (dalam topologi) tercatat dalam log pada setiap UML yang memiliki aturan drop.
```
(Untuk SURABAYA)
iptables -N LOGGING
iptables -A FORWARD -p tcp --dport 22 -d 10.151.83.80/29 -i eth0 -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level info
iptables -A LOGGING -j DROP

(Untuk MALANG dan MOJOKERTO)
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: " --log-level info
iptables -A LOGGING -j DROP
```  
Penjelasan :
- Buat table iptables baru untuk LOGGING.
- Menggubah iptables sebelumnya yang memiliki rule DROP menjadi LOGGING.
- Menggunakan LOGGING untuk menset agar mencatat log info paket yang di DROP dengan prefix catatan `IPTables-Dropped: `.
- Setelah selesai log, paket akan di DROP.
