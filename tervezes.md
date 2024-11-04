# Saját cég bemutatása
Az ITWorks informatikai szolgáltatásokat nyújt széles körben, főleg hálózatok kiépítését, de más szolgáltatásokat is.
# Az ügyfél
A Cacti.hu egy növények értékesítésével foglalkozó kisvállalkozás, és arra kért fel bennünket, építsünk ki nekik egy biztonságos, és hatékony hálózatot új irodájukba.
# A helyszín
Az épületben található egy váró és lobby is, valamint 3 külön iroda és 1 szerverszoba is.
![Az iroda alaprajza](KomarL_Iroda.PNG)
# Igényfelmérés
Az alábbi táblázat tartalmazza a részletes Címzési tervet.

# VLANOK
A cég összes részlege külön VLAN-ba került, így könnyebben menedzselhetőek
NUMBER|NAME|
-|-|
10|MANAGEMENT
20|VEZETOSEG
30|HR
40|MARKETING
Az alábbihoz hasonlóan jártunk el a többi VLAN esetében is.
```
SW1(config)#vlan 10
SW1(config-vlan)#name MANAGEMENT
```
Valamint a konfiguráció megkönnyítése érdekében VTP-t is használtunk mindhárom kapcsoló esetében

A VTP szerver SW1, Cacti.hu a domain, és a jelszó cacti, valamint VTP 2-es verziót használunk
```
SW1(config)#vtp mode server
SW1(config)#vtp domain cacti.hu
SW1(config)#vtp password cacti
SW1(config)#vtp version 2
```
A táblázatban láthtó
# STP
A hurkok elkerülése érdekében feszítőfa protokollt alkalmaztunk az összekötött kapcsolókon.

Rapid-pvst-t alkalmaztunk a 10-es, 20-as, 30-as, és 40-es vlanokon és mindegyiknél SW1 a root, valamint PortFastot is beállítottunk, így az interfész egy pillanat alatt elérhetővé válik.
# Etherchannel
Redundancia és terheléselosztás céljából etherchannelt is implementáltunk az összes kapcsoló között.
# DHCP

# Telnet/SSH

# HSRP

# Biztonság
## Portbiztonság

## Jelszavak titkosítása

## DHCP Snooping

## DAI

## BPDU Guard

# Forgalomirányítás

# NAT, Portátirányítás

# WiFi

# Server

# GRE
