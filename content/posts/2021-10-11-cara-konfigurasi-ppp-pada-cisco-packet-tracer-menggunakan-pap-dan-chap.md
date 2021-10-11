---
author: Aidil Putra
date: '2021-10-11'
slug: cara-konfigurasi-ppp-pada-cisco-packet-tracer-menggunakan-pap-dan-chap
tags:
- "cisco packet tracer"
- ppp
- chap
- pap
- ospf
title: "Cara Konfigurasi ppp pada Cisco Packet Tracer menggunakan pap dan chap"
type: post
---

{{< figure src="/assets/images/2021-10-11-cisco-ppp-pap-chap.png" alt="Cisco packet tracer" position="center" >}}

Artikel ini akan membahas bagaimana mengkonfigurasi ppp pada `cisco packet tracer` menggunakan authentikasi `pap` dan `chap`
dan sebagai tambahan koneksi koneksi yang melewati network yang bebeda akan di cover oleh routing `ospf`.

## Skema
* r1 Se2/0 10.0.0.1/8, Fa0/0 192.168.10.0/24
* r2 Se2/0 10.0.0.2/8, Se3/0 20.0.0.2/8
* r3 Se3/0 20.0.0.1/8, Fa0/0, 192.168.20.0/24
* pc-aidil-1 192.168.10.2/24
* pc-aidil-2 192.168.20.2/24

## Detail
* Koneksi dari r1 ke r2 menggunakan authentikasi `pap` (clear text)
* Koneksi dari r3 ke r2 menggunakan authentikasi `chap` (encrypted text)
* Koneksi antar router menggunakan kabel `serial DCE`
* Koneksi dari router ke switch dan dari switch ke pc menggunakan kabel `Straight`
* Switch menggunakan mode access (vlan 1)
* Password pap "ppp-pap"
* Password chap "ppp-chap"

Dari tabel diatas kita akan menerapkan 2 tipe authentikasi PPP r1 se2/0 –> r2 se2/0 dengan authentikasi PAP dan r2 se3/0 –> r3 se3/0 dengan authentikasi CHAP.
Setelah koneksi PPP berhasil dibuat, yang perlu dilakukan adalah melakukan routing (ospf) agar antar client yang berbeda network bisa saling terhubung.

## Konfigurasi
Untuk mengkonfigurasi router silahkan klik icon router dan masuk ke menu `CLI` dan ikuti langkah berikut.
```bash
#
# Konfigurasi router r1
#
Router>enable 
Router#configure terminal
Router(config)#hostname r1
r1(config)#username r2 password ppp-pap
r1(config)#interface se2/0
r1(config-if)#ip address 10.0.0.1 255.0.0.0
r1(config-if)#encapsulation ppp
r1(config-if)#ppp authentication pap
r1(config-if)#ppp pap sent-username r1 password ppp-pap
r1(config-if)#no shutdown
r1(config-if)#exit
r1(config)#interface fastEthernet 0/0
r1(config-if)#ip address 192.168.10.1 255.255.255.0
r1(config-if)#no shutdown
r1(config-if)#exit
r1(config)#
```

Keluar dan masuk ke router r2
```consile
#
# Konfigurasi router r2
#
Router>enable
Router#configure terminal
Router(config)#hostname r2
r2(config)#username r1 password ppp-pap
r2(config)#username r3 password ppp-chap
r2(config)#interface se2/0
r2(config-if)#ip address 10.0.0.2 255.0.0.0
r2(config-if)#clock rate 64000
r2(config-if)#encapsulation ppp
r2(config-if)#ppp authentication pap
r2(config-if)#ppp pap sent-username r2 password ppp-pap
r2(config-if)#no shutdown
r2(config-if)#exit
r2(config)#interface serial 3/0
r2(config-if)#ip address 20.0.0.2 255.0.0.0
r2(config-if)#encapsulation ppp 
r2(config-if)#ppp authentication chap 
r2(config-if)#no shutdown
r2(config-if)#exit
r2(config)#
```

Keluar dan masuk ke router r3
```bash
Router>enable
Router#configure terminal
Router(config)#hostname r3
r3(config)#username r2 password ppp-chap
r3(config)#interface se3/0
r3(config-if)#ip address 20.0.0.1 255.0.0.0
r3(config-if)#clock rate 64000
r3(config-if)#encapsulation ppp
r3(config-if)#ppp authentication chap 
r3(config-if)#no shutdown
r3(config-if)#exit
r3(config)#interface fastEthernet 0/0
r3(config-if)#ip address 192.168.20.1 255.255.255.0
r3(config-if)#no shutdown
r3(config-if)#exit
r3(config)#
```

Sampai disini konfigurasi PPP sudah selesai dan tinggal melakukan verifikasi apakah konfigurasi PPP kita sudah berhasil.. Untuk memastikan protocol encapsulationnya sudah menjadi PPP silahkan masuk ke router dan lihat status interface nya.
```bash
r1>show interfaces se2/0
Serial2/0 is up, line protocol is up (connected)
  Hardware is HD64570
  Internet address is 10.0.0.1/8
  MTU 1500 bytes, BW 128 Kbit, DLY 20000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation PPP, loopback not set, keepalive set (10 sec)
  LCP Open
  Open: IPCP, CDPCP
  Last input never, output never, output hang never
```

Selanjutnya kita akan mengkonfigurasi routing `ospf` agar pc-aidil-1 (192.168.10.2/24) dan pc-aidil-2 (192.168.20.2) bisa saling terhubung.

```bash
#
# Konfigruasi router r1
#
r1(config)#router ospf 10
r1(config-router)#network 10.0.0.0 255.255.255.0 area 0
r1(config-router)#network 192.168.10.0 0.0.0.255 area 0
r1(config-router)#log-adjacency-changes 
r1(config-router)#exit
r1(config)#exit
r1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration…
```

Keluar dan masuk ke router r2
```bash
#
# Konfigruasi router r2
#
r2(config)#router ospf 10
r2(config-router)#network 10.0.0.0 255.255.255.0 area 0
r2(config-router)#network 20.0.0.0 255.255.255.0 area 0
r2(config-router)#log-adjacency-changes 
r2(config-router)#exit
r2(config)#exit
r2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration…
```

Keluar dan masuk ke router r3
```bash
#
# Konfigruasi router r2
#
r3(config)#router ospf 10
r3(config-router)#network 20.0.0.0 255.255.255.0 area 0
r3(config-router)#network 192.168.20.0 0.0.0.255 area 0
r3(config-router)#log-adjacency-changes 
r3(config-router)#exit
r3(config)#exit
r3#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration…
```

Untuk routing `ospf` telah selesai di konfigurasi, kita bisa melakukan pengecekan dengan cara masuk ke router r1 dan lihat apakah prefix router r2 (192.168.20.0/24) telah dimiliki oleh routerr1

```bash
r1>show ip route  | include 192.168.20
O    192.168.20.0/24 [110/129] via 10.0.0.2, 00:23:23, Serial2/0
```
Dilihat dari tabel routing di atas saat ini router r1 telah memiliki prefx 192.168.20.0/24 dengan flag `O` dimana arti nya prefix tersebut di dapat dari routing dinamis `ospf` yang arti nya konfigurasi `ospf` telah berhasil.

Terakhir tinggal pastikan bahwa pc-aidil-1 sudah bisa melakukan ping ke pc-aidil-2 dengan cara masuk ke `command prompt`
```bash
C:\>ipconfig

FastEthernet0 Connection:(default port)
   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::202:4AFF:FE32:1362
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.10.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 192.168.10.1


C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Reply from 192.168.20.1: bytes=32 time=2ms TTL=253
Reply from 192.168.20.1: bytes=32 time=5ms TTL=253
Reply from 192.168.20.1: bytes=32 time=4ms TTL=253
Reply from 192.168.20.1: bytes=32 time=5ms TTL=253

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 2ms, Maximum = 5ms, Average = 4ms
```


