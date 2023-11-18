# Jarkom-Modul-3-E06-2023
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

#### ping testing pada client

![ping](https://github.com/Hfdrsyd/Jarkom-Modul-3-E06-2023/blob/main/images/ping.png)


## Bagian 1
Berjalannya waktu, petualang diminta untuk melakukan deployment.
1. Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3. (6)
2. Kepala suku dari Bredt Region memberikan resource server sebagai berikut:
Lawine, 4GB, 2vCPU, dan 80 GB SSD.
Linie, 2GB, 2vCPU, dan 50 GB SSD.
Lugner 1GB, 1vCPU, dan 25 GB SSD.
aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000 request dan 100 request/second. (7)
3. Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut:
- Nama Algoritma Load Balancer
- Report hasil testing pada Apache Benchmark
- Grafik request per second untuk masing masing algoritma. 
- Analisis (8)
4. Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire. (9)
5. Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/ (10)
6. Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id. (11) hint: (proxy_pass)
7. Selanjutnya LB ini hanya boleh diakses oleh client dengan IP [Prefix IP].3.69, [Prefix IP].3.70, [Prefix IP].4.167, dan [Prefix IP].4.168. (12) hint: (fixed in dulu clinetnya)


#### (6) pada setiap php worker (Lawine, Linie, Lugner) setting sebagai berikut untuk deployment
```
apt-get update && apt install nginx php php-fpm -y git
git config --global http.sslVerify false
git clone https://github.com/Hfdrsyd/Jarkom-Modul-3-E06-2023.git

mkdir /var/www/granz
cp -r /Jarkom-Modul-3-E06-2023/granz.channel.yyy.com/modul-3 /var/www/granz

gg='server {

        listen 80;

        root /var/www/granz/modul-3;

        index index.php index.html index.htm;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        }

        location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/jarkom_error.log;
        access_log /var/log/nginx/jarkom_access.log;
 }'


echo "$gg" > /etc/nginx/sites-available/jarkom

rm -rf /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/jarkom /etc/nginx/sites-enabled
service php7.3-fpm start
service nginx restart
```

Berikut lynx pada salah satu worker (Linie) di client

![lynx](images/lynx.png)

#### (7) lakukan setting pada load balancer (Eisen)
setting dilakukan menggunakan algoritma Round Robin namun menggunakan weight sehingga dapat seolah-olah merepresentasikan resource server.
```
apt-get update
apt-get install apache2-utils -y

mkdir -p /etc/nginx/rahasisakita
htpasswd -cb /etc/nginx/rahasisakita/htpasswd netics ajkE06

ff='upstream granz{
    server 192.209.3.1;
    server 192.209.3.2;
    server 192.209.3.3;
}

upstream riegel{
    server 192.209.4.1;
    server 192.209.4.2;
    server 192.209.4.3;
}

server {
    listen 81;
    server_name _; # Change to your actual domain

    location / {
        auth_basic "Restricted Content";
        auth_basic_user_file /etc/nginx/rahasisakita/htpasswd;
        proxy_pass http://granz;
    }
}

server {
    listen 80;
    server_name _; # Change to your Laravel domain

    location / {
        proxy_pass http://riegel;
    }
}'
unlink /etc/nginx/sites-enabled/default
echo "$ff" > /etc/nginx/sites-available/lb-switch4
ln -sf /etc/nginx/sites-available/lb-switch4 /etc/nginx/sites-enabled/lb-switch4
```
![hasil lynx](nanti diubah)

#### testing dengan 1000 request dan 100 request/second.
```
ab -n 1000 -c 100 -g out.txt http://granz.channel.e06.com/
```
![htop](nanti diubah)

![ab](nanti diubah)

### (8) dan (9)
#### <a href="https://docs.google.com/document/d/1YD0pExQ0IW7_EknqfYw6FCfXo7eCfq0nbOAaxD0O1EU/edit?usp=sharing">Silahkan buka Grimore disini untuk melihat hasil </a> 

#### (10)



