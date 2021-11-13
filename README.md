# Jarkom-Modul-3-IUP3-2021

#### Members name:
* 05111942000025	Gilbert Kurniawan H
* 05111942000004	Rafi Akbbar Rafsanjani
* 05111942000020	Drigo Alexander Sihombing

## no. 1

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

At number 1 we must first install EniesLobby as DNS Server, Jipangu as DHCP server and Water7 as proxy Server.
```
// EniesLobby
apt-get update
apt-get install bind9 -y
```

```
// Jipangu
apt-get update
apt-get install isc-dhcp-server -y
```

```
// in Water7
apt-get update
apt-get install squid -y
```

After that we have to set up the DHCP server so that we can connect to Foosha as mentioned in Problem 2.
```
// Jipangu
INTERFACES="eth0"
```

## no. 2

Foosha sebagai DHCP Relay.

We set the  Foosha as DHCP relay, the first thing we have to do is install
```
// Foosha
apt-get update
apt-get install isc-dhcp-relay -y
```

Then we install by using ```apt-get install isc-dhcp-relay -y```. It will be some settings that we need to input 
```
10.39.2.4
eth1 eth2 eth3
```

```10.39.2.4``` it means the Ip of our DHCP server Jipangu, ```eth1 eth2 eth3``` it means the interfaces ou DHCP relay that connect to switch 1,2,3.



## no. 3

Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 and [prefix IP].1.150 - [prefix IP].1.169

First we need to setting the subnet in the DHCP server. We edit the file in ```/etc/dhcp/dhcpd.conf```

```
subnet 10.39.1.0 netmask 255.255.255.0 {
    range 10.39.1.20 10.39.1.99;
    range 10.39.1.150 10.39.1.169;
    option routers 10.39.1.1;
    option broadcast-address 10.39.1.255;
    option domain-name-servers 10.39.2.2;
    default-lease-time 360;
    max-lease-time 7200;
}

subnet 10.39.3.0 netmask 255.255.255.0 {
    range 10.39.3.30 10.39.3.50;
    option routers 10.39.3.1;
    option broadcast-address 10.39.3.255;
    option domain-name-servers 10.39.2.2;
    default-lease-time 720;
    max-lease-time 7200;
}

subnet 10.39.2.0 netmask 255.255.255.0 {
        option routers 10.39.2.1;
}
```
At this problem we change the switch 1 (10.39.1.0) range to ```0.39.1.20 until 10.39.1.99``` and ```10.39.1.150 until 10.39.1.169```


## no. 4

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

## no. 5

Client mendapatkan DNS dari EniesLobby and client dapat terhubung dengan internet melalui DNS tersebut.

## no. 6

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit seandgkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

## no. 7

Luffy and Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

## no. 8

Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Jawab

#### Water7

We open `/etc/squid/squid.conf` and edit with

```bash
    http_port 5000
    visible_hostname jualbelikapal.iup3.com
    http_access allow all
```

![8.1](imgs/8.1.JPG)

then, we can run command `service squid restart`

#### Skypie

we first run `apt-get update` and `apt-get install lynx`

We activate the proxy with command `export http_proxy="http://10.39.2.3:5000"`. We also run command `env | grep -i proxy`to check if the proxy already running.

![8.2](imgs/8.2.JPG)

## no. 9

Agar transaksi jual beli lebih aman and pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy and zorobelikapalyyy dengan password zoro_yyy

### Jawab

#### Water7

We first run `apt-get update` and `apt-get install apache2-utils`

We then run command `htpasswd -cm /etc/squid/passwd luffybelikapaliup3`, option `c` used to create a new file, while `m` used so that we use MD5 encryption. We also input `luffy_iup3` as the password.

Then, we run `htpasswd -m /etc/squid/passwd zorobelikapaliup3`. After that we input `zoro_iup3` as the password.
![9.1](imgs/9.1.JPG)

We also edit file `/etc/squid/squid.conf` into

```bash
    http_port 5000
    visible_hostname jualbelikapal.iup3.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
```

![9.2](imgs/9.2.JPG)

We then run command `service squid restart`

## no. 10

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 and setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

### Jawab

#### Water7

Create a new file named `acl.conf` inside squid folder by typing `nano /etc/squid/acl.conf`. Inside the file, type in:

```bash
    acl AVAILABLE_WORKING time MTWH 07:00-11:00
    acl AVAILABLE_WORKING time TWHF 17:00-23:59
    acl AVAILABLE_WORKING time WHFA 00:00-03:00
```

![10.1](imgs/10.1.JPG)

After that, we also edit `/etc/squid/squid.conf` into

```bash
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.iup3.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS AVAILABLE_WORKING
    http_access deny all
```

![10.2](imgs/10.2.JPG)

After that, run the command `service squid restart`

## no. 11

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

### Jawab

#### EniesLobby

Edit file `nano /etc/bind/named.conf.local` by typing

```bash
    zone "super.franky.iup3.com" {
        type master;
        file "/etc/bind/sunnygo/super.franky.iup3.com";
    };

    zone "3.39.10.in-addr.arpa" {
        type master;
        file "/etc/bind/sunnygo/3.39.10.in-addr.arpa";
    };
```

![11.1](imgs/11.1.JPG)

Create a folder name `sunnygo` using command `mkdir /etc/bind/sunnygo`. 

After that, type command `cp /etc/bind/db.local /etc/bind/sunnygo/super.franky.iup3.com`. After that, edit file `nano /etc/bind/sunnygo/super.franky.iup3.com` by filling in

```bash
    $TTL    604800
    @       IN      SOA     super.franky.iup3.com. root.super.franky.iup3.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      super.franky.iup3.com.
    @       IN      A       10.39.3.69 ; IP Skypie
    www     IN      CNAME   super.franky.iup3.com.
```

![11.2](imgs/11.2.JPG)

After that, run the command `cp /etc/bind/db.local /etc/bind/sunnygo/3.39.10.in-addr.arpa`. After that, edit file `nano /etc/bind/sunnygo/3.39.10.in-addr.arpa` by typing

```bash
    $TTL    604800
    @       IN      SOA     super.franky.iup3.com. root.super.franky.iup3.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
    ;
    3.39.10.in-addr.arpa.      IN      NS      super.franky.iup3.com.
    69       IN      PTR       super.franky.iup3.com.
```

![11.3](imgs/11.3.JPG)

Then, we run command `service bind9 restart`

#### Skypie

We first install `apache2`, `php`, `libapache2-mod-php7.0`, `wget`, and `unzip`.

Then we can run command `wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip` then we unzip by typing `unzip super.franky.zip`

After that, we move to directory `/etc/apache2/sites-available`. Then we copy file `000-default.conf` into `super.franky.iup3.com.conf`

![11.4](imgs/11.4.JPG)

Then, we setting file `super.franky.iup3.com.conf` by adding line 

```bash
	ServerName super.franky.iup3.com
	ServerAlias www.super.franky.iup3.com
	DocumentRoot /var/www/super.franky.iup3.com
```

![11.5](imgs/11.5.JPG)

We then create another directory named `super.franky.iup3.com` on `/var/www/` by using command `mkdir /var/www/super.franky.iup3.com`. Then we copy from the unzipped `super.franky` folder into `/var/www/super.franky.iup3.com`.

After that, we run command `a2ensite super.franky.iup3.com` and `service apache2 restart`

![11.6](imgs/11.6.JPG)

#### Water7

After that, edit file `/etc/squid/squid.conf` into

```bash
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.iup3.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    acl google dstdomain www.google.com
    http_access allow USERS AVAILABLE_WORKING
    http_access deny all google
    deny_info http://super.franky.iup3.com/ google
    http_access deny all
```

![11.7](imgs/11.7.JPG)

Then, we run command `service squid restart`
