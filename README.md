# Jarkom Modul 2 | IT29 | 2023
- M. Azril Fathoni (5027211002)
- Ridho Husni Indrawan (5027211043)
## Soal 1
> Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut.
### Topologi
![image](https://github.com/0xazr/Jarkom-Modul-2-IT29-2023/assets/54212814/fa982fec-fb19-4170-83ab-b220b2a0d451)
### Network Configuration
#### Router
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.78.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.78.2.1
	netmask 255.255.255.0
```
#### Arjuna (Load Balancer)
```
auto eth0
iface eth0 inet static
	address 10.78.1.4
	netmask 255.255.255.0
	gateway 10.78.1.1
```
#### Yudhistira (DNS Master Server)
```
auto eth0
iface eth0 inet static
	address 10.78.1.2
	netmask 255.255.255.0
	gateway 10.78.1.1
```
#### Werkudara (DNS Slave Server)
```
auto eth0
iface eth0 inet static
	address 10.78.1.3
	netmask 255.255.255.0
	gateway 10.78.1.1
```
#### Abimanyu (Web Servers)
```
auto eth0
iface eth0 inet static
	address 10.78.2.4
	netmask 255.255.255.0
	gateway 10.78.2.1
```
#### Prabukusuma (Web Server)
```
auto eth0
iface eth0 inet static
	address 10.78.2.3
	netmask 255.255.255.0
	gateway 10.78.2.1
```
#### Wisanggeni (Web Server)
```
auto eth0
iface eth0 inet static
	address 10.78.2.2
	netmask 255.255.255.0
	gateway 10.78.2.1
```
#### Sadewa (Client)
```
auto eth0
iface eth0 inet static
	address 10.78.2.5
	netmask 255.255.255.0
	gateway 10.78.2.1
```
#### Nakula (Client)
```
auto eth0
iface eth0 inet static
	address 10.78.2.6
	netmask 255.255.255.0
	gateway 10.78.2.1
```
### Enable Internet Access
Agar setiap node dalam jaringan dapat mengakses internet melalui NAT, maka digunakan command berikut pada Router:
```bash
# enable all nodes to access internet
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.78.0.0/16
```
Selain itu, juga diperlukan untuk men-set nameserver pada setiap node dengan menggunakan bash script seperti berikut:
```bash
# define the nameserver for internet access
nameserver="192.168.122.1"

# add the nameserver to resolv.conf
if [ -f /etc/resolv.conf ]; then
    if ! grep -q "nameserver $nameserver" /etc/resolv.conf; then
        echo "nameserver $nameserver" | tee -a /etc/resolv.conf > /dev/null
    fi
fi
```
## Soal 2
> Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.
### Setup DNS for arjuna.it29.com (Yudhistira)
1. Install depedencies
```bash
# install dependencies
apt-get update -y
apt-get install bind9 -y
```
2. Start bind9
```bash
service bind9 start
```
3. Pada file `/etc/bind/named.local.conf` tambahkan line berikut:
```
zone "arjuna.it29.com" {
    type master;
    file "/etc/bind/it29/arjuna.it29.com";
};
```
4. Buat DNS recordnya pada `/etc/bind/it29/arjuna.it29.com`:
```
$TTL    604800
@       IN      SOA     arjuna.it29.com. root.arjuna.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.it29.com.
@       IN      A       10.78.1.4
www     IN      CNAME   arjuna.it29.com.
```
5. Restart bind9 service
```
service bind9 restart
```
## Soal 3
> Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.
### Setup DNS for abimanyu.it29.com (Yudhistira)
1. Pada file `/etc/bind/named.local.conf` tambahkan line berikut:
```
zone "abimanyu.it29.com" {
    type master;
    file "/etc/bind/it29/abimanyu.it29.com";
};
```
2. Buat DNS recordnya pada `/etc/bind/it29/abimanyu.it29.com`.
```
$TTL    604800
@       IN      SOA     abimanyu.it29.com. root.abimanyu.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      abimanyu.it29.com.
@           IN      A       10.78.2.4
www         IN      CNAME   abimanyu.it29.com.
```
3. Restart bind9 service
```
service bind9 restart
```
## Soal 4
> Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
### Setup DNS for parikesit.abimanyu.it29.com (Yudhistira)
1. Pada file `/etc/bind/named.local.conf` tambahkan line berikut:
```
zone "parikesit.abimanyu.it29.com" {
    type master;
    file "/etc/bind/it29/baratayuda/parikesit.abimanyu.it29.com";
};
```
2. Buat DNS recordnya pada file `/etc/bind/it29/baratayuda/parikesit.abimanyu.it29.com`.
```
$TTL    604800
@       IN      SOA     parikesit.abimanyu.it29.com. root.parikesit.abimanyu.it29.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      parikesit.abimanyu.it29.com.
@       IN      A       10.78.2.4
www     IN      CNAME   parikesit.abimanyu.it29.com.
```
3. Restart bind9 service
```
service bind9 restart
```
## Soal 5
> Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)
### Setup Reverse DNS for abimanyu.it29.com (Yudhistira)
1. Pada file `/etc/bind/named.conf.local` tambahkan line berikut:
```
zone "2.78.10.in-addr.arpa" {
    type master;
    file "/etc/bind/it29/2.78.10.in-addr.arpa";
};
```
2. Buat DNS recordnya pada file `/etc/bind/it29/2.78.10.in-addr.arpa`.
```
$TTL    604800
@       IN      SOA     abimanyu.it29.com. root.abimanyu.it29.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.78.10.in-addr.arpa.   IN      NS      abimanyu.it29.com.
4                       IN      PTR     abimanyu.it29.com.
```
3. Restart bind9 service
```
service bind9 restart
```
## Soal 6
> Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
### Setup DNS for arjuna.it29.com & abimanyu.it29.com on DNS Master (Yudhistira)
1. Edit konfigurasi zone arjuna.it29.com & abimanyu.it29.com pada file `/etc/bind/named.local.conf` sehingga menjadi seperti berikut. 
```
zone "arjuna.it29.com" {
    type master;
    notify yes;
    also-notify { 10.78.1.3; };
    allow-transfer { 10.78.1.3; };
    file "/etc/bind/it29/arjuna.it29.com";
};

zone "abimanyu.it29.com" {
    type master;
    notify yes;
    also-notify { 10.78.1.3; };
    allow-transfer { 10.78.1.3; };
    file "/etc/bind/it29/abimanyu.it29.com";
};
```
2. Restart bind9 service
```
service bind9 restart
```
### Setup DNS for arjuna.it29.com & abimanyu.it29.com on DNS Slave (Werkudara)
1. Install depedencies
```
# install dependencies
apt-get update -y
apt-get install bind9 -y
```
2. Start bind9 service
```
service bind9 start
```
3. Pada file `/etc/bind/named.local.conf` tambahkan line berikut:
```
zone "arjuna.it29.com" {
    type slave;
    masters { 10.78.1.2; };
    file "/etc/bind/it29/arjuna.it29.com";
};

zone "abimanyu.it29.com" {
    type slave;
    masters { 10.78.1.2; };
    file "/etc/bind/it29/abimanyu.it29.com";
};
```
4. Buat DNS record untuk kedua domain. Berikut adalah DNS record untuk domain arjuna.it29.com pada file `/etc/bind/it29/arjuna.it29.com`.
```
$TTL    604800
@       IN      SOA     arjuna.it29.com. root.arjuna.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.it29.com.
@       IN      A       10.78.1.4
www     IN      CNAME   arjuna.it29.com.
```
dan berikut adalah DNS record untuk domain abimanyu.it29.com pada file `/etc/bind/it29/abimanyu.it29.com`.
```
$TTL    604800
@       IN      SOA     arjuna.it29.com. root.arjuna.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.it29.com.
@       IN      A       10.78.1.4
www     IN      CNAME   arjuna.it29.com.
```
5. Restart bind9 service
```
service bind9 restart
```
## Soal 7
> Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
### Setup DNS for baratayuda.abimanyu.it29.com on DNS Master (Yudhistira)
1. Edit file `/etc/bind/it29/abimanyu.it29.com` seperti berikut:
```
$TTL    604800
@       IN      SOA     abimanyu.it29.com. root.abimanyu.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      abimanyu.it29.com.
@           IN      A       10.78.2.4
www         IN      CNAME   abimanyu.it29.com.
ns1         IN      A       10.78.1.3   ; IP Werkudara
baratayuda  IN      NS      ns1
```
2. Edit juga file `/etc/bind/named.conf.options`.
```
options {
        directory "/var/cache/bind";

        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
3. Restart bind9 service
```
service bind9 restart
```
### Setup DNS for baratayuda.abimanyu.it29.com on DNS Slave (Werkudara)
1. Pada file `/etc/bind/named.local.conf` tambahkan line seperti berikut:
```
zone "baratayuda.abimanyu.it29.com" {
    type master;
    file "/etc/bind/it29/baratayuda/baratayuda.abimanyu.it29.com";
};
```
2. Buat DNS record pada file `/etc/bind/it29/baratayuda/baratayuda.abimanyu.it29.com`.
```
$TTL    604800
@       IN      SOA     baratayuda.abimanyu.it29.com. root.baratayuda.abimanyu.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      baratayuda.abimanyu.it29.com.
@           IN      A       10.78.2.4
www         IN      CNAME   baratayuda.abimanyu.it29.com.
```
3. Restart bind9 service
```
service bind9 restart
```
## Soal 8
> Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.
### Setup DNS for rjp.baratayuda.abimanyu.it29.com on DNS Slave (Werkudara)
1. Pada file `/etc/bind/named.local.conf` tambahkan line seperti berikut:
```
zone "rjp.baratayuda.abimanyu.it29.com" {
    type master;
    file "/etc/bind/it29/baratayuda/rjp.baratayuda.abimanyu.it29.com";
};
```
2. Buat DNS recordnya pada file `/etc/bind/it29/baratayuda/rjp.baratayuda.abimanyu.it29.com`.
```
$TTL    604800
@       IN      SOA     rjp.baratayuda.abimanyu.it29.com. root.rjp.baratayuda.abimanyu.it29.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@           IN      NS      rjp.baratayuda.abimanyu.it29.com.
@           IN      A       10.78.2.4
www         IN      CNAME   rjp.baratayuda.abimanyu.it29.com.
```
3. Restart bind9 service
```
service bind9 restart
```
## Soal 9 & Soal 10
> Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
### Setup Load Balancer (Arjuna)
1. Install depedencies
```
# install dependencies
apt-get update -y
apt-get install nginx -y
```
2. Edit file `/etc/nginx/nginx.conf` menjadi seperti berikut:
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

http {
    upstream nodes_lb {
        server 10.78.2.2:8003;
        server 10.78.2.3:8001;
        server 10.78.2.4:8002;
    }

    server {
        listen 80;
        listen [::]:80;

        server_name arjuna.it29.com;

        location / {
            proxy_pass http://nodes_lb;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```
3. Start nginx service
```
service nginx start
```
### Setup Load Balancer Worker (Prabukusuma, Wisanggeni, Abimanyu)
1. Install depedencies pada semua worker
```
# install dependencies
apt-get update -y
apt-get install nginx unzip php7.2-fpm lsb-release ca-certificates apt-transport-https software-properties-common -y
```
2. Konfigurasi web server pada file `/etc/nginx/sites-available/arjuna.it29.com` untuk setiap worker:
- Prabakusuma
```
server {
    listen 8001;
    listen [::]:8001;

    root /var/www/arjuna.it29.com;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name arjuna.it29.com;

    location / {
        try_files \$uri \$uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
- Wisanggeni
```
server {
    listen 8003;
    listen [::]:8003;

    root /var/www/arjuna.it29.com;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name arjuna.it29.com;

    location / {
        try_files \$uri \$uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
- Abimanyu
```
server {
    listen 8002;
    listen [::]:8002;

    root /var/www/arjuna.it29.com;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name arjuna.it29.com;

    location / {
        try_files \$uri \$uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
3. Lakukan symlink pada file konfigurasi tadi dengan menjalankan command berikut pada setiap worker:
```bash
# symlink nginx configuration
ln -s /etc/nginx/sites-available/arjuna.it29.com /etc/nginx/sites-enabled/arjuna.it29.com
```
4. Lakukan deployment resource yang sudah diberikan pada setiap worker:
```bash
# create directory for arjuna.it29.com
mkdir -p /var/www/arjuna.it29.com

# wget dist.zip
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX' -O /tmp/dist.zip

# unzip dist.zip
unzip /tmp/dist.zip -d /tmp

# move dist to /var/www/arjuna.it29.com
mv /tmp/arjuna.yyy.com/index.php /var/www/arjuna.it29.com/index.php
```
5. Start php dan nginx service
```
# start php
service php7.2-fpm start

# restart nginx
service nginx restart
```
## Soal 11
> Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server www.abimanyu.yyy.com. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy
### Setup web server for abimanyu.it29.com (Abimanyu)
1. Install depedencies
```bash
apt-get install apache2 libapache2-mod-php7.2 -y
```
2. Setup environtment
Hal berikut dilakukan agar abimanyu.it29.com dapat berjalan pada port 80.
```bash
# remove default nginx configuration
rm /etc/nginx/sites-enabled/default
rm /etc/nginx/sites-available/default
```
3. Buat konfigurasi web server pada file `/etc/apache2/sites-available/abimanyu.it29.com.conf`
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/abimanyu.it29
    ServerName abimanyu.it29.com
    ServerAlias www.abimanyu.it29.com
    
    <Directory /var/www/abimanyu.it29>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
4. Enable konfigurasi abimanyu.it29.com.conf dengan command berikut:
```bash
# a2ensite abimanyu.it29.com
a2ensite abimanyu.it29.com
```
5. Deployment resource
```bash
# create directory for abimanyu.it29.com
mkdir -p /var/www/abimanyu.it29

# wget abimanyu_dist.zip
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1a4V23hwK9S7hQEDEcv9FL14UkkrHc-Zc' -O /tmp/abimanyu_dist.zip

# unzip abimanyu_dist.zip --force
unzip -o /tmp/abimanyu_dist.zip -d /tmp

# move dist to /var/www/abimanyu.it29
mv /tmp/abimanyu.yyy.com/* /var/www/abimanyu.it29/ -f
```
6. Disable default config apache2
```
# disable default apache2 configuration
a2dissite 000-default.conf
```
7. Start Apache2 service
```bash
service apache2 start
```
## Soal 12
> Setelah itu ubahlah agar url www.abimanyu.yyy.com/index.php/home menjadi www.abimanyu.yyy.com/home.
### Change /home to /index.php/home (Abimanyu)
1. Enable module rewrite pada apache2
```
# activate rewrite module
a2enmod rewrite

# restart apache2
service apache2 restart
```
2. Buat file /var/www/abimanyu.it29/.htaccess dengan isi sebagai berikut:
```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^home$ /index.php/home [NC,L]
```
## Soal 13
> Selain itu, pada subdomain www.parikesit.abimanyu.yyy.com, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy
### Setup web server for parikesit.abimanyu.it29.com (Abimanyu)
1. Buat konfigurasi web server pada file `/etc/apache2/sites-available/parikesit.abimanyu.it29.com.conf`
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/parikesit.abimanyu.it29
    ServerName parikesit.abimanyu.it29.com
    ServerAlias www.parikesit.abimanyu.it29.com

    <Directory /var/www/parikesit.abimanyu.it29>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>

    Alias "/js" "/var/www/parikesit.abimanyu.it29/public/js"

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
2. Enable konfigurasi abimanyu.it29.com.conf dengan command berikut:
```bash
# a2ensite parikesit.abimanyu.it29.com
a2ensite parikesit.abimanyu.it29.com
```
3. Deployment resource
```bash
# create directory for parikesit.abimanyu.it29.com
mkdir -p /var/www/parikesit.abimanyu.it29

# wget parikesit_dist.zip
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1LdbYntiYVF_NVNgJis1GLCLPEGyIOreS' -O /tmp/parikesit_dist.zip

# unzip parikesit_dist.zip --force
unzip -o /tmp/parikesit_dist.zip -d /tmp

# move dist to /var/www/parikesit.abimanyu.it29
mv /tmp/parikesit.abimanyu.yyy.com/* /var/www/parikesit.abimanyu.it29/ -f
```
4. Restart Apache2 service
```bash
service apache2 restart
```
## Soal 14
> Pada subdomain tersebut folder /public hanya dapat melakukan directory listing sedangkan pada folder /secret tidak dapat diakses (403 Forbidden).
### Set Forbidden Rule for /secret directory (Abimanyu)
1. Buat folder /secret dan isi dengan test file
```
# add secret/ folder to parikesit.abimanyu.it29
mkdir -p /var/www/parikesit.abimanyu.it29/secret
echo "SECRET FILE!!!" > /var/www/parikesit.abimanyu.it29/secret/secret.txt
```
2. Buat .htaccess pada `/var/www/parikesit.abimanyu.it29/secret/.htaccess`
```
Options -Indexes
```
## Soal 15
