# Jarkom-Modul-2-E06-2023
Berikut adalah repository dari kelompok E06 untuk pengerjaan Praktikum Modul 3 Jaringan Komputer. Repository ini akan berisikan dokumentasi cara pengerjaan tiap soal, screenshot output, dan kendala yang dialami.

# Anggota Kelompok
| Nama | NRP | 
| --- | --- |
| Muhammad Hafidh Rosyadi | 5025211013 |
| Kartika Diva Asmara Gita | 5025211039 |

# Dokumentasi Pengerjaan Soal
## Bagian 1
Setelah mengalahkan Demon King, perjalanan berlanjut. Kali ini, kalian diminta untuk melakukan register domain berupa riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP **(0)** mengarah pada worker yang memiliki IP [prefix IP].x.1.

**(1)** Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.

Kemudian, karena masih banyak spell yang harus dikumpulkan, bantulah para petualang untuk memenuhi kriteria berikut.:
Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80 **(2)**

Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168 **(3)**

Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut **(4)**

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit **(5)**

#### Topologi (1)
![Topologi](https://github.com/Hfdrsyd/Jarkom-Modul-3-E06-2023/blob/main/images/topo.png)

#### Setting pada Himmel/DHCP Server
```
apt-get update
apt-get install isc-dhcp-server -y

a='INTERFACESv4="eth0"
INTERFACESv6=""'

echo "$a" > /etc/default/isc-dhcp-server


config='ddns-update-style none;
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

authoritative;
log-facility local7;

subnet 192.209.1.0 netmask 255.255.255.0 {
  option routers 192.209.1.1;
}

subnet 192.209.2.0 netmask 255.255.255.0 {
  option routers 192.209.2.1;
}

subnet 192.209.3.0 netmask 255.255.255.0 {
    # (2)
    range 192.209.3.16 192.209.3.32;
    range 192.209.3.64 192.209.3.80;
    option routers 192.209.3.4;
    option broadcast-address 192.209.3.255;
    option domain-name-servers 192.209.1.3;
    # (5)
    default-lease-time 180;
    max-lease-time 5760;
}

subnet 192.209.4.0 netmask 255.255.255.0 {
    # (3)
    range 192.209.4.12 192.209.4.20;
    range 192.209.4.160 192.209.4.168;
    option routers 192.209.4.4;
    option broadcast-address 192.209.4.255;
    option domain-name-servers 192.209.1.3;
    # (5)
    default-lease-time 720;
    max-lease-time 5760;
}

host Lawine {
    hardware ethernet 8a:d5:50:e4:b5:66;
    fixed-address 192.209.3.1;
}

host Linie {
    hardware ethernet 92:2a:32:38:62:fe;
    fixed-address 192.209.3.2;
}

host Lugner {
    hardware ethernet c2:21:bb:45:de:b9;
    fixed-address 192.209.3.3;
}

host Frieren {
    hardware ethernet 96:b8:1f:4e:bf:fb;
    fixed-address 192.209.4.1;
}

host Flamme {
    hardware ethernet d2:59:48:28:a2:d4;
    fixed-address 192.209.4.2;
}

host Fern {
    hardware ethernet 86:ec:9f:90:81:6d;
    fixed-address 192.209.4.3;
}'


echo "$config" > /etc/dhcp/dhcpd.conf

rm /var/run/dhcpd.pid

service isc-dhcp-server restart
```

#### Setting DHCP Relay aura

```
apt-get update
apt-get install isc-dhcp-relay -y
service isc-dhcp-relay start


a='SERVERS="192.209.1.2"
INTERFACES="eth1 eth2 eth3 eth4"
OPTIONS='

echo "$a" > /etc/default/isc-dhcp-relay

b='net.ipv4.ip_forward=1'

echo "$b" > /etc/sysctl.conf

service isc-dhcp-relay restart
```
#### Setting DNS pada DNS server heiter
```
apt-get update
apt-get install bind9 -y

mkdir /etc/bind/granz
mkdir /etc/bind/riegel
# (0)
a='zone "granz.channel.E06.com" {
    type master;
    file "/etc/bind/granz/granz.channel.E06.com";
};

zone "riegel.canyon.E06.com" {
    type master;
    file "/etc/bind/riegel/riegel.canyon.E06.com";
};'

echo "$a" > /etc/bind/named.conf.local


b=';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     riegel.canyon.E06.com. root.riegel.canyon.E06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      riegel.canyon.E06.com.
@           IN      A       192.209.4.1 ; IP Frieren
www         IN      CNAME   riegel.canyon.E06.com.'

echo "$b" > /etc/bind/riegel/riegel.canyon.E06.com


c=';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     granz.channel.E06.com. root.granz.channel.E06.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      granz.channel.E06.com.
@           IN      A       192.209.3.1 ; IP Lawine
www         IN      CNAME   granz.channel.E06.com.'

echo "$c" > /etc/bind/granz/granz.channel.E06.com
# (4)
d='options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };
        allow-query{any;};

        //dnssec-validation auto;

        listen-on-v6 { any; };
};'

echo "$d" > /etc/bind/named.conf.options
service bind9 restart
```


