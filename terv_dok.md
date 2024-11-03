# Kik is vagyunk mi
Cégem az ITWorks informatikai szolgáltatásokat nyújt széles körben, főleg hálózatok kiépítését, de más szolgáltatásokat is szoktunk nyújtani, például kérésre vállaljuk weboldalak készítését is.
# Az ügyfél
A Cacti.hu egy növények értékesítésével foglalkozó kisvállalkozás, és arra kért fel bennünket, építsünk ki nekik egy biztonságos, és hatékony hálózatot új irodájukba.
# A helyszín
Az épületben található egy váró és lobby is, valamint 3 külön iroda és 1 szerverszoba is.
# Igényfelmérés
A cég kért egy tárterületet, ahova alkalmazottai menthetik fájljaikat, levelező- és web szolgáltatásokat.
# Részlegek és területeik kezelése
Mivel több különálló részleg van a cégnél, közel egymáshoz VLAN-okat használtunk, valamint ezek konfigurációjának megkönnyítése érdekében VTP-t is.

Először is beállítjuk a kapcsolón a VTP-t:
```
SW1(config)#vtp domain cacti.hu
Changing VTP domain name from NULL to cacti.hu
SW1(config)#vtp password cacti
Setting device VLAN database password to cacti
```
Ezután mivel VTP szerver alapértelmezetten, létrehozzuk rajta a VLAN-okat:
```
SW1(config)#vlan 10
SW1(config-vlan)#name MANAGEMENT
SW1(config-vlan)#vlan 20
SW1(config-vlan)#name VEZETOSEG
SW1(config-vlan)#vlan 30
SW1(config-vlan)#name HR
SW1(config-vlan)#vlan 40
SW1(config-vlan)#name MARKETING
```
Majd pedig hozzárendeljük őket a portokhoz:
```
SW1(config-vlan)#interface range FastEthernet 0/1-2
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 10
SW1(config-if-range)#interface GigabitEthernet 0/1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport nonegotiate
SW1(config-if)#switchport trunk allowed vlan 10,20,30,40
SW1(config-if)#switchport trunk native vlan 99
```
A többi kapcsolót kliensként konfiguráljuk:
```
SW2(config)#vtp mode client
Setting device to VTP CLIENT mode.
SW2(config)#vtp domain cacti.hu
Changing VTP domain name from NULL to cacti.hu
SW2(config)#vtp password cacti
Setting device VLAN database password to cacti
```
Majd itt is hozzárendeljük a portokat:
```
SW2(config)#interface range FastEthernet 0/1-2
SW2(config-if-range)#switchport mode access
SW2(config-if-range)#switchport nonegotiate
SW2(config-if-range)#switchport access vlan 40
```
Ezt le is ellenörizhetjük:
```
SW1#show vlan brief
...
10   MANAGEMENT                       active    
20   VEZETOSEG                        active    
30   HR                               active    
40   MARKETING                        active    Fa0/1, Fa0/2
...
```
A harmadik kapcsolón is elvégezzü ugyanezen lépéseket.

Ezután beállítjuk az alinterfészeket a forgalomirányítón
```
R1(config)#interface GigabitEthernet 0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#interface GigabitEthernet 0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 192.168.1.65 255.255.255.192
R1(config-subif)#interface GigabitEthernet 0/0.30
R1(config-subif)#encapsulation dot1Q 30
R1(config-subif)#ip address 192.168.1.129 255.255.255.192
R1(config-subif)#interface GigabitEthernet 0/0.40
R1(config-subif)#encapsulation dot1Q 40
R1(config-subif)#ip address 192.168.1.193 255.255.255.192
```
# A magas sebesség és rendelkezésre állás biztosítása
A hurkok elkerülése érdekében feszítőfa protokollt alkalmaztunk.  
A következőket kiadjuk mindhárom kapcsolón
```
SW1(config)#spanning-tree mode rapid-pvst
SW1(config)#spanning-tree vlan 10,20,30,40
SW1(config)#spanning-tree portfast default
SW1(config)#spanning-tree portfast bpduguard default
```
SW1 esetében kiadjuk ezt a plusz parancsot hogy ő legyen e root
```
SW1(config)#spanning-tree vlan 10,20,30,40 root primary
```
Le lehet ellenőrizni:
```
SW1#show spanning-tree summary 
Switch is in rapid-pvst mode
Root bridge for: VLAN0010 VLAN0020 VLAN0030 VLAN0040
Extended system ID           is enabled
Portfast Default             is enabled
PortFast BPDU Guard Default  is enabled
Portfast BPDU Filter Default is disabled
Loopguard Default            is disabled
EtherChannel misconfig guard is disabled
UplinkFast                   is disabled
BackboneFast                 is disabled
Configured Pathcost method used is short

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     2         0        0          1          3
VLAN0010                     0         0        0          3          3
VLAN0020                     1         0        0          2          3
VLAN0030                     1         0        0          2          3
VLAN0040                     1         0        0          2          3

---------------------- -------- --------- -------- ---------- ----------
5 vlans                      5         0        0         10         15
```
Redundancia és terheléselosztás céljából etherchannelt is implementáltunk  
Mivel a konfiguráció ehhez hasonló, csak ezt az egyet mutatom be.
```
SW1(config)#interface range FastEthernet 0/21-22
SW1(config-if-range)#channel-group 1 mode active
Creating a port-channel interface Port-channel 1
SW1(config-if-range)#interface range FastEthernet 0/23-24
SW1(config-if-range)#channel-group 2 mode active
Creating a port-channel interface Port-channel 2
SW1(config-if-range)#interface port-channel 1
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport nonegotiate
SW1(config-if)#switchport trunk native vlan 99
SW1(config-if)#switchport trunk allowed vlan 10,20,30,40
SW1(config-if)#interface port-channel 2
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport nonegotiate
SW1(config-if)#switchport trunk native vlan 99
SW1(config-if)#switchport trunk allowed vlan 10,20,30,40
```
Természetesen ezt is leellenőrizhetjük a következő paranccsal:
```
SW1#show etherchannel summary
...
Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           LACP   Fa0/21(P) Fa0/22(P) 
2      Po2(SU)           LACP   Fa0/23(P) Fa0/24(P) 
```
# Hálozati konfiguráció
Mivel azt kérték, hogy minden eszközről el lehessen érni az internetet, ezért DHCP szolgáltatást is konfiguráltunk.

Kizárjuk a címeket a nyomtatók és kapcsolók számára, majd beállítjuk a kiosztandó címeket és persze a DNS szerver címét is hozzáadjuk.
```
R1(config)#ip dhcp excluded-address 192.168.1.2 192.168.1.5
R1(config)#ip dhcp excluded-address 192.168.1.66 192.168.1.69
R1(config)#ip dhcp excluded-address 192.168.1.130 192.168.1.133
R1(config)#ip dhcp excluded-address 192.168.1.194 192.168.1.197
R1(config)#ip dhcp pool lan10
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#dns-server 192.168.11.11
R1(dhcp-config)#ip dhcp pool lan20
R1(dhcp-config)#network 192.168.1.64 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.65
R1(dhcp-config)#dns-server 192.168.11.11
R1(dhcp-config)#ip dhcp pool lan30
R1(dhcp-config)#network 192.168.1.128 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.129
R1(dhcp-config)#dns-server 192.168.11.11
R1(dhcp-config)#ip dhcp pool lan40
R1(dhcp-config)#network 192.168.1.192 255.255.255.192
R1(dhcp-config)#default-router 192.168.1.193
R1(dhcp-config)#dns-server 192.168.11.11
```
Hasonlóképpen járunk el R0 esetén is, ahol a vezeték nélküli eszközök számára is biztosítjuk az internetelérést
Majd a már jól megszokott ellenőrzés:
```
R1#show ip dhcp pool

Pool public_wifi :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 0
 Excluded addresses             : 5
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.0.1          192.168.0.1      - 192.168.0.254     0    / 5     / 254

Pool lan10 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 5
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      0    / 5     / 62

Pool lan20 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 5
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.65         192.168.1.65     - 192.168.1.126     0    / 5     / 62

Pool lan30 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 5
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.129        192.168.1.129    - 192.168.1.190     0    / 5     / 62

Pool lan40 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 5
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.193        192.168.1.193    - 192.168.1.254     0    / 5     / 62
```
# Távoli Adminisztráció
A cég azt kérte, rendszergazdáik könnyedén hozzáférhessenek a hálózati eszközökhöz.
A fokozott biztonság érdekében a nem megbízható forrásból is elérhető eszközöket SSH kapcsolattal védjük, a belső kapcsolók esetében viszont Telnet protokollt használunk a nagyobb sebesség elérése érdekében.

Telnet konfiguráció SW1 kapcsolón
```
SW1(config)#ip default-gateway 192.168.1.1
SW1(config)#interface vlan 10
SW1(config-if)#ip address 192.168.1.3 255.255.255.192
SW1(config-if)#no shutdown
SW1(config-if)#interface vlan 20
SW1(config-if)#ip address 192.168.1.67 255.255.255.192
SW1(config-if)#no shutdown
SW1(config-if)#interface vlan 30
SW1(config-if)#ip address 192.168.1.131 255.255.255.192
SW1(config-if)#no shutdown
SW1(config-if)#interface vlan 40
SW1(config-if)#ip address 192.168.1.195 255.255.255.192
SW1(config-if)#no shutdown
SW1(config-if)#username user1 password pass1
SW1(config)#line vty 0 15
SW1(config-line)#login local
```
SSH beállítása SW4 kapcsolón
```
SW4(config)#ip domain-name cacti.hu
SW4(config)#crypto key generate rsa
SW4(config)#ip ssh version 2
SW4(config)#ip ssh authentication-retries 0
SW4(config)#username user1 password pass1
SW4(config)#line vty 0 15
SW4(config-line)#login local
SW4(config-line)#transport input ssh
```
Mivel kapcsolóval van dolgunk kell neki adni IP-címet is
```
SW4(config-line)#interface vlan 1
SW4(config-if)#ip address 192.168.0.2 255.255.255.0
SW4(config-if)#no shutdown
SW4(config-if)#exit
SW4(config)#ip default-gateway 192.168.0.1
```
A többi eszközön is hasonló dolgunk van
Az ellenőrzés:
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
...
Vlan1                  192.168.0.2     YES manual up                    down
R1#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 0
```
# A szerver magas rendelkezésre állása
Hogy a szervert mindig el lehessen érni redundáns átjárót alkalmaztunk, így ha az egyik kiesne, a másik átveszi szerepét.

HSRP konfiguráció R1-en
```
R1(config)#interface GigabitEthernet 0/1
R1(config-if)#standby version 2
R1(config-if)#standby ip 192.168.11.9
```
Ugyanezt megismételjük R2-n is.
# Biztonság
## Portok biztosítása
A DTP letiltása, valamint a nem használt portoknak a letiltása és egy nem használt VLAN-ba való helyezése segít lecsökkenteni a váratlan támadások esélyét
```
SW1(config)#interface range FastEthernet 0/1, GigabitEthernet 0/1-2
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport nonegotiate
SW1(config-if-range)#interface range FastEthernet 0/2-24
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport nonegotiate
SW1(config-if-range)#switchport access vlan 100
SW1(config-if-range)#shutdown
```
De persze ha már portokról beszélünk, a klasszikus portbiztonság sem maradhat le, úgyhogy konfiguráljuk azt is gyorsan:
```
SW1(config)#interface range FastEthernet 0/1-20, GigabitEthernet 0/2
SW1(config-if-range)#switchport port-security
SW1(config-if-range)#switchport port-security mac-address sticky
SW1(config-if-range)#switchport port-security violation restrict
```
Így a kapcsoló megjegyzi az első MAC-címet amit lát, és ha következönek valaki más próbál csatlakozni, azt feljegyzi
## Jelszavak biztosítása
Állítsunk be egy titkos jelszót, hogy ne léphessen be bárki, majd titkosítsuk az összes jelszót az eszközön
```
SW1(config)#enable secret cacti
SW1(config)#service password-encryption
```
## DHCP biztonsága
A portbiztonság hasznos, de az sem véd mindentől, így a DHCP hamisítás ellen külön védelmet kell alkalmazni.
Állítsunk be minden VLAN-ra dhcp snoopingot, hogy csak a megbízható eszközök felől érkezhessen DHCP
```
SW1(config)#ip dhcp snooping
SW1(config)#ip dhcp snooping vlan 1,10,20,30,40,99,100
SW1(config)#interface GigabitEthernet 0/1
SW1(config-if)#ip dhcp snooping trust
```
## ARP tábla biztonsága
De mivel a hackerek is egyre ügyesebbek, és semmi sem véd mindent egyszerre, külön DAI-t is kell alkalmaznunk.
```
SW1(config-if)#ip arp inspection vlan 1,10,20,30,40,99,100
SW1(config)#ip arp inspection validate dst-mac ip src-mac
```
## STP biztonsága
Hogy megakadájozzuk, hogy ellenőrizetlen kapcsolókat csatoljanak a rendszerre, BpduGuardot alkamaztunk
# A hálózat hatékony működésének biztosítása
A skálázhatóság, hatékonyság és biztonság érdekében dinamikus OSPF alapú forgalomirányítást alkalmaztunk, MD5 hitelesítéssel.

Az internetre egy alapértelmezett átjáró vezet
```
R0(config)#ip route 0.0.0.0 0.0.0.0 Serial 0/1/0
```
```
R0(config)#router ospf 1
R0(config-router)#network 192.168.11.8 0.0.0.7 a 0
R0(config-router)#network 192.168.11.4 0.0.0.3 a 0
R0(config-router)#passive-interface default
R0(config-router)#no passive-interface Serial 0/0/1
R0(config-router)#interface Serial 0/0/1
R0(config-if)#ip ospf authentication message-digest
R0(config-if)#ip ospf message-digest-key 1 md5 cacti
```
# Internetelérés

```
R0(config)#interface range GigabitEthernet 0/0-2
R0(config-if-range)#ip nat inside
R0(config-if-range)#interface Serial 0/0/0
R0(config-if)#ip nat inside
R0(config-if)#interface Serial 0/0/1
R0(config-if)#ip nat outside
R0(config-if)#ip access-list standard NAT_ACL
R0(config-std-nacl)#permit any
R0(config-std-nacl)#ip nat pool NAT_PL 10.0.0.10 10.0.0.14 netmask 255.255.255.248
R0(config)#ip nat inside source list NAT_ACL pool NAT_PL overload
R0(config)#ip nat inside source static tcp 192.168.11.11 80 10.0.0.10 80
R0(config)#ip nat inside source static tcp 192.168.11.11 443 10.0.0.10 443
R0(config)#ip nat inside source static tcp 192.168.11.11 53 10.0.0.10 53
R0(config)#ip nat inside source static udp 192.168.11.11 53 10.0.0.10 53
```
# WiFi
A vezeték nélküli eszközök megkapják az összes szükséges információt a hálózat eléréséhez
- IP
- Gateway
- DNS

public_wifi - (guest access)  
192.168.0.0/24

privat  
192.168.10.0/24  
SSID - Cacti_Wifi  
Broadcast disabled  
WPA2PSK AES Cacti_Pass123  
Admin password - Cacti_Admin
# Szerver szolgáltatások
- konfiguráció mentése tftp-n
- ftp a fájlok tárolásához - ftp.cacti.hu
- céges levelező szerver - mail.cacti.hu
- saját webszerver - cacti.hu
- dns szerver az ftp- és levelezőszerverek eléréséhez
# Távoli hálózat közvetlen elérése
A cég kérésére közvetlen kapcsolatot építettünk ki egy másik irodájukkal
```
R0(config)#interface tunnel 0
R0(config-if)#ip address 192.168.2.1 255.255.255.252
R0(config-if)#tunnel source Serial 0/1/0
R0(config-if)#tunnel destination 3.3.3.2

RR(config)#interface tunnel 0
RR(config-if)#ip address 192.168.2.2 255.255.255.252
RR(config-if)#tunnel source Serial 0/0/0
RR(config-if)#tunnel destination 1.1.1.2
RR(config-if)#exit
RR(config-if)#ip route 0.0.0.0 0.0.0.0 Serial 0/0/0
RR(config)#router ospf 1
RR(config-router)#network 192.168.2.0 0.0.0.3 a 0
RR(config-router)#network 172.16.0.0 0.0.0.3 a 0
```