

Kelompok B02
Name	NRP
```
Muhammad Daffa Aldriantama (5025201177)
Burhanudin Rifa (5025201191)
Maisan Auliya (5025201137)
```
IP Prefix Kelompok: 10.4

Soal dan Jawaban

## Topologi

![Alt text](/img/g1.png)

Eden adalah DNS Server
WISE adalah DHCP Server
Garden dan SSS adalah Web Server
Jumlah Host pada Forger adalah 62 host
Jumlah Host pada Desmond adalah 700 host
Jumlah Host pada Blackbell adalah 255 host
Jumlah Host pada Briar adalah 200 host

## VLSM
Dalam mengerjakan soal ini, kami menggunakan teknik VLSM dengan menggunakan GNS3.

Pembagian Subnet

![Alt text](/img/g2.png)

Hal pertama yang dilakukan adalah menentukan subnet-subnet pada topologi. Pembagian subnet berdasarkan setiap node yang terhubung melalui switch ataupun tidak, dalam satu daerah.


Penentuan Jumlah IP
Setelah itu, setiap subnet akan dihitung jumlah IP yang dibutuhkan dan netmask yang dihasilkan berdasarkan node-node yang terhubung.


VLSM Tree
Setelah mendapatkan jumlah IP setiap subnet, akan dibuat Tree berdasarkan topologi. Pada subnet induk memiliki NID 10.4.0.0 dengan netmask /21 sehingga pembagian IP dilakukan berdasarkan NID dan Netmask yang didapat hingga mencapai subnet terbawah.

iamge

Lalu, kita mendapatkan IP untuk masing-masing subnet.


Eth Configuration
Pada masing-masing node akan dilakukan network configuration, kita melakukan assignment dengan IP yang telah ditetapkan sesuai dengan subnet

Strix

Karena menggunakan DHCP, pada Strix dibuat fix address dengan memasukkan hwaddress agar IP tidak berubah-ubah.
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
hwaddress ether b6:23:6b:b5:5d:33

auto eth1
iface eth1 inet static
address 10.4.7.149
netmask 255.255.255.252

auto eth2
iface eth2 inet static
address 10.4.7.145
netmask 255.255.255.252
```
Ostania
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.4.7.150
netmask 255.255.255.252
gateway 10.4.7.149

auto eth1
iface eth1 inet static
address 10.4.4.1
netmask 255.255.254.0

auto eth2
iface eth2 inet static
address 10.4.7.137
netmask 255.255.255.248

auto eth3
iface eth3 inet static
address 10.4.6.1
netmask 255.255.255.0
```
Westalis
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.4.7.146
netmask 255.255.255.252
gateway 10.4.7.145

auto eth1
iface eth1 inet static
address 10.4.0.1
netmask 255.255.252.0

auto eth2
iface eth2 inet static
address 10.4.7.1
netmask 255.255.255.128

auto eth3
iface eth3 inet static
address 10.4.7.129
netmask 255.255.255.248
```
Garden
```
auto eth0
iface eth0 inet static
address 10.4.7.138
netmask 255.255.255.248
gateway 10.4.7.137
SSS

auto eth0
iface eth0 inet static
address 10.4.7.139
netmask 255.255.255.248
gateway 10.4.7.137
```
Eden
```
auto eth0
iface eth0 inet static
address 10.4.7.130
netmask 255.255.255.248
gateway 10.4.7.129
```
WISE
```
auto eth0
iface eth0 inet static
address 10.4.7.131
netmask 255.255.255.248
gateway 10.4.7.129
```
Blackbell, Briar, Desmond, dan Forger sebagai Client sehingga akan dibuat eth configurationnya sebagai berikut.
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```
Routing
Setelah itu akan dilakukan routing pada GNS3 agar semua router, server, dan client akan saling terhubung.

Strix
```
route add -net 10.4.7.128 netmask 255.255.255.248 gw 10.4.7.146
route add -net 10.4.7.0 netmask 255.255.255.128 gw 10.4.7.146
route add -net 10.4.0.0 netmask 255.255.252.0 gw 10.4.7.146

route add -net 10.4.4.0 netmask 255.255.254.0 gw 10.4.7.150
route add -net 10.4.6.0 netmask 255.255.255.0 gw 10.4.7.150
route add -net 10.4.7.136 netmask 255.255.255.248 gw 10.4.7.150
```
## Config DNS Server
Node yang berperan sebagai DNS Server yaitu Eden akan menambahkan konfigurasi pada file /etc/bind/named.conf.options sebagai berikut.

options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                192.168.122.1;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
## Config DHCP Server
Node yang berperan sebagai DHCP Server yaitu WISE akan menambahkan konfigurasi pada file-file sebagai berikut.

.bashrc
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install isc-dhcp-server -y
```
/etc/default/isc-dhcp-server

```
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts

#
# This is a POSIX shell fragment
#

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```
/etc/dhcp/dhcpd.conf
```
#Forger
subnet 10.4.7.0 netmask 255.255.255.128 {
    range 10.4.7.2 10.4.7.126;
    option routers 10.4.7.1;
    option broadcast-address 10.4.7.127;
    option domain-name-servers 10.4.7.130; 
    default-lease-time 600;
    max-lease-time 7200;
}

#Desmond
subnet 10.4.0.0 netmask 255.255.252.0 {
    range 10.4.0.2 10.4.3.254;
    option routers 10.4.0.1;
    option broadcast-address 10.4.3.255;
    option domain-name-servers 10.4.7.130;
    default-lease-time 600;
    max-lease-time 7200;
}

#Blackbell
subnet 10.4.4.0 netmask 255.255.254.0 {
    range 10.4.4.2 10.4.5.254;
    option routers 10.4.4.1;
    option broadcast-address 10.4.5.255;
    option domain-name-servers 10.4.7.130;
    default-lease-time 600;
    max-lease-time 7200;
}

#Briar
subnet 10.4.6.0 netmask 255.255.255.0 {
    range 10.4.6.2 10.4.6.254;
    option routers 10.4.6.1;
    option broadcast-address 10.4.6.255;
    option domain-name-servers 10.4.7.130;
    default-lease-time 600;
    max-lease-time 7200;
}

subnet 10.4.7.128 netmask 255.255.255.248 {
    option routers 10.4.7.129;
}
```
## Config DHCP Relay
Node yang berperan sebagai DHCP Relay yaitu Ostania dan Westalis akan menambahkan konfigurasi pada file-file sebagai berikut.

.bashrc
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
echo "" | apt-get install isc-dhcp-relay -y
```
/etc/default/isc-dhcp-relay
```
# What servers should the DHCP relay forward requests to?
SERVERS="10.4.7.131"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth0 eth1 eth2 eth3"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
## Config Web Server
Node yang berperan sebagai Web Server yaitu Garden dan SSS akan menambahkan konfigurasi pada file-file sebagai berikut.

.bashrc
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install apache2 -y
```
/var/www/html/index.html
```
# Garden
Garden nih

# SSS
SSS nih
```
/etc/apache2/ports.conf
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 443

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
## Penyelesaian
1
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Strix menggunakan iptables, tetapi Loid tidak ingin menggunakan MASQUERADE

Pertama kita dapatkan terlebih dahulu IP dari Strix yang berhubungan dengan NAT dengan menggunakan command ip a.


Karena diminta untuk tidak menggunakan MASQUERADE, maka digunakan SNAT. Source akan diubah dari yang awalnya 0.0 ke Strix dengan --to-source 192.168.122.213.

```
iptables -t nat -A POSTROUTING -s 10.4.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.213
```


2
Kalian diminta untuk melakukan drop semua TCP dan UDP dari luar Topologi kalian pada server yang merupakan DHCP Server demi menjaga keamanan

Syntax iptables yang dapat digunakan untuk masalah tersebut adalah sebagai berikut. Iptables ditempatkan pada WISE yang merupakan DHCP server.
```
iptables -A INPUT ! -s 10.4.0.0/21 -p tcp -j DROP
iptables -A INPUT ! -s 10.4.0.0/21 -p udp -j DROP
```
Ketika terdapat node yang mengakses dari luar topologi, semua yang mengakses secara tcp atau udp akan di drop. 


3
Loid meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 2 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop

Karena koneksinya ICMP maka akan digunakan flag -p dengan nilai icmp. Selain, itu akan digunakan limit maksimal 2 untuk akses koneksi secara bersamaan sehingga dapat menggunakan --connlimit-above 2 dan menambahkan target DROP agar koneksi lainnya selain 2 koneksi tersebut ditolak.
```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 2 --connlimit-mask 0 -j DROP
```

4
Akses menuju Web Server hanya diperbolehkan disaat jam kerja yaitu Senin sampai Jumat pada pukul 07.00 - 16.00

Karena dibatasi waktu tertentu maka kita batasi menggunakan -m time dengan --timestart sebagai waktu mulai dapat diakses bernilai 07.00 dan --timestop sebagai waktu akhir dapat diakses bernilai 16.00, serta membatasi hari berlaku dengan --weekdays untuk hari Senin hingga Jumat. Terdapat flag -j yang menentukan kapan akses akan di ACCEPT atau REJECT.

Pada Node Eden (DNS Server)
```
iptables -A INPUT -s 10.4.7.136/29 -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -s 10.4.7.136/29 -j REJECT
```
Pada Node Garden dan SSS (Web Server)
```
iptables -A INPUT -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
iptables -A INPUT -j REJECT
```

5
Karena kita memiliki 2 Web Server, Loid ingin Ostania diatur sehingga setiap request dari client yang mengakses Garden dengan port 80 akan didistribusikan secara bergantian pada SSS dan Garden secara berurutan dan request dari client yang mengakses SSS dengan port 443 akan didistribusikan secara bergantian pada Garden dan SSS secara berurutan

Karena diminta untuk setiap request akan didistribusikan secara bergantian antara Garden dan SSS, maka akan dikonfigurasi untuk masing-masing node dengan port untuk request masing-masing node adalah 80 dan 443 dengan menggunakan --dport. Selain itu, akan dibatasi secara bergantian dengan menggunakan --every 2 sehingga akan bergantian terdistribusinya dengan mengarahkan ke node lain dengan menggunakan --to-destination.
```
iptables -A PREROUTING -t nat -p tcp --dport 80 -d 10.4.7.138 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.4.7.138:80
iptables -A PREROUTING -t nat -p tcp --dport 80 -d 10.4.7.138 -j DNAT --to-destination 10.4.7.139:80
iptables -A PREROUTING -t nat -p tcp --dport 443 -d 10.4.7.139 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.4.7.139:443
iptables -A PREROUTING -t nat -p tcp --dport 443 -d 10.4.7.139 -j DNAT --to-destination 10.4.7.138:443
```

6
Karena Loid ingin tau paket apa saja yang di-drop, maka di setiap node server dan router ditambahkan logging paket yang di-drop dengan standard syslog level

Pada Wise restart terlebih dahulu DHCP nya dengan:
```
service isc-dhcp-server restart
service isc-dhcp-server restart
```
kemudian isikan sylog syntax pada wise untuk melihat paket yang di drop .
```
iptables -N LOGGING
iptables -A INPUT -p icmp -m connlimit --connlimit-above 2 --connlimit-mask 0 -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IPTables-Dropped: "
iptables -A LOGGING -j DROP
```
kemudian isikan sylog pada semua node
```
service apache2 restart
service apache2 restart

iptables -A INPUT -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -N LOGGING
iptables -A INPUT -j LOGGING
iptables -A LOGGING -j LOG --log-prefix "IPTables-Rejected: "
iptables -A LOGGING -j REJECT

service rsyslog restart
```


