## Praktikum Modul 2
Kelompok D13
- 05111840000094 Rafi Nizar Abiyyi
- 05111740000192 Faishal Abiyyudzakir

<br>

### Soal 1
> membuat topologi jaringan

![](/imgs/no1-1.PNG)

hasil topologi.sh

```
# D13

# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.57 eth1=daemon,,,switch1 eth2=daemon,,,switch2 eth3=daemon,,,switch3 mem=256M &

# Server switch 2
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

# Klien switch 1
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &

# Klien switch 3
xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &

```

<br>

### Soal 2

> Surabaya ditunjuk sebagai perantara DHCP relay antara DHCP server dan klien

install isc-dhcp-relay di UML **SURABAYA**

![](/imgs/no2-1.PNG)

Setelah install isc-dhcp-relay akan muncul prompt yang meminta IP tujuan forward dan interface yang menerima requests

Relay akan meneruskan request ke **TUBAN** dengan ip `10.151.79.116` dari interface eth1, eth2, dan eth3.

![](/imgs/no2-2.PNG)

Config relay di **TUBAN**, subnet DMZ menerima dari IP **SURABAYA**

```
subnet 10.151.79.112 netmask 255.255.255.248 {
    option routers 10.151.79.113;
}
```

![](/imgs/no2-4.PNG)

DHCP **MOJOKERTO**, menerima fixed-address `10.151.79.115`

![](/imgs/no2-5.PNG)

DHCP **MALANG**, menerima fixed-address `10.151.79.114`

![](/imgs/no2-6.PNG)

DHCP **GRESIK** (subnet 1), menerima address `192.168.0.10`

![](/imgs/no2-7.PNG)

DHCP **SIDARJO** (subnet 1), menerima address `192.168.0.11`

![](/imgs/no2-8.PNG)

DHCP **BANYUWANGI** (subnet 3), menerima address `192.168.1.51`

![](/imgs/no2-9.PNG)

DHCP **MADIUN** (subnet 4), menerima address `192.168.1.50`

<br>

### Soal 3
> client subnet 1 memiliki range ip 192.168.0.10-192.168.0.100 dan range 192.168.0.110-192.168.0.200

![](/imgs/no3-1.PNG)

Menggunakan range untuk subnet `192.168.0.0`
```
range 192.168.0.10 192.168.0.100
range 192.168.0.110 192.168.0.200
```

<br>

### Soal 4
> client subnet 3 memiliki range ip 192.168.1.50-192.168.1.70

![](/imgs/no4-1.PNG)

Menggunakan range untuk subnet `192.168.1.0`

```
range 192.168.1.50 192.168.1.70
```

<br>

### Soal 5
> client mendapatkan DNS malang dan DNS 202.46.129.2 dari DHCP

![](/imgs/no5-1.PNG)

Menggunakan option DNS untuk kedua subnet

```
option domain-name-servers 202.46.129.2, 10.151.79.114
```

<br>

### Soal 6
> client subnet 1 mendapatkan lease time 5 menit dan client subnet 3 mendapat lease time 10 menit


![](/imgs/no6-1.PNG)

Menggunakan lease time 300 detik pada subnet 1

```
default-lease-time 300;
max-lease-time 7200;
```

Menggunakan lease time 300 detik pada subnet 3

```
default-lease-time 600;
max-lease-time 7200;
```

<br>

### Soal 7
> User autentikasi milik anri memiliki format ```userta_d13:inipassw0rdta_d13```

Menginstall squid dan apache-utils di UML **MOJOKERTO**

![](/imgs/no7-1.PNG)

Menggunakan apache-utils membuat krendensial login baru

```
htpasswd -c /etc/squid/passwd userta_d13
```

Membuat user dengan nama `userta_d13` dan passwod `inipassw0rdta_d13`

![](/imgs/no7-2.PNG)

Membuat rule auth di `/etc/squid/squid.conf`

```
auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
http_access allow USERS
```

![](/imgs/no7-3.PNG)

Prompt autentikasi

<br>

### Soal 8
> setiap hari Selasa-Rabu pukul 13.00-18.00. Bu Meguri membatasi penggunaan internet Anri hanya pada jadwal yang telah ditentukan itu saja. Maka diluar jam tersebut, Anri tidak dapat mengakses jaringan internet dengan proxy tersebut

![](/imgs/no8-1.PNG)

Membuat `/etc/squid/acl.conf` yang berisi
```
acl KERJA_TA time TW 13:00-18:00
```

![](/imgs/no8-2.PNG)

Pada `/etc/squid/squid.conf` tambah include file acl tersebut

```
include /etc/squid/acl.conf
```
dan izinkan `USERS` untuk time `KERJA_TA`
```
http_access allow USERS KERJA_TA
```

<br>

### Soal 9
> setiap hari Selasa-Kamis pukul 21.00 - 09.00 keesokan harinya (sampai Jumat jam 09.00)

![](/imgs/no9-1.PNG)

Pada `/etc/squid/acl.conf` tambah time
```
acl BIMBINGAN_AWAL time TWH 21:00-23:59
acl BIMBINGAN_AKHIR time WHF 00:00-09:00
```


![](/imgs/no9-2.PNG)

Pada `/etc/squid/squid.conf` tambah include file acl tersebut

```
include /etc/squid/acl.conf
```
dan izinkan `USERS` untuk time `KERJA_TA`
```
http_access allow USERS BIMBINGAN_AWAL
http_access allow USERS BIMBINGAN_AKHIR
```
<br>

### Soal 10
> redirect http://google.com ke http://monta.if.its.ac.id

![](/imgs/no10-1.PNG)

Membuat `/etc/squid/ban.conf` yang berisi
```
google.com
```

![](/imgs/no10-2.PNG)

Pada `/etc/squid/squid.conf` tambah include file `ban.acl` tersebut

```
acl NOPE url_regex "/etc/squid/ban.acl"
```

deny akses url pada list dalam `ban.acl` dan redirect menuju http://monta.if.its.ac.id/

```
deny_info http://monta.if.its.ac.id/ NOPE
http_access deny NOPE
```

<br>

### Soal 11
> mengubah default error Proxy

![](/imgs/no11-1.PNG)

download error page di `/usr/share/squid/errors` lalu extract

```
tar -xvf error403.tar.gz
```

![](/imgs/no11-2.PNG)

isi folder error403/ dengan isi dari templates/ dan overwrite html error 403

```
cp -r templates/* error403/
cp -i error403/error403.html error403/ERR_ACCESS_DENIED
```

![](/imgs/no11-6.PNG)

Pada `/etc/squid/squid.conf` tambah error directory

```
error_directory /usr/share/squid/errors/error403
```

![](/imgs/no11-7.PNG)

Hasil deny dari **MOJOKERTO**

<br>

### Soal 12
> Karena Bu Meguri dan Anri adalah tipe orang pelupa, maka untuk memudahkan mereka, Anri memiliki ide ketika menggunakan proxy cukup dengan mengetikkan domain janganlupa-ta.yyy.pw dan memasukkan port 8080


![](/imgs/no12-1.PNG)

Pada UML **MALANG** install bind9 dan buat zone baru di `/etc/bind/named.conf`

```
zone "janganlupa-ta.d13.pw" {
    type "master";
    file "/etc/bind/jarkom/janganlupa-ta.d13.pw";
};
```

![](/imgs/no12-2.PNG)

Pada `/etc/bind/jarkom/janganlupa-ta.d13.pw` tambah record A yang mengarah ke IP **MOJOKERTO**

```
@   IN  A   10.151.79.115
```

![](/imgs/no12-3.PNG)

Menggunakan proxy `janganlupa-ta.d13.pw`

<br>

akhir dari lapres.