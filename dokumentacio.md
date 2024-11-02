# A megrendelésről
A Cacti.hu egy dísznövények értékesítésével foglalkozó kisvállalkozás.  
Cégem az ITWorks informatikai szolgáltatásokat nyújt széles körben.  
Első lépésként meghallgattuk a vevő igényeit, majd felmértük a környezetet.

# A hálózatról
Az irodában dolgozó személyek számára biztonságos és hatékony internetkapcsolat biztosítása.  

## Vezeték nélküli kapcsolat
A vevő igényeinek megfelelően a mobileszközök számára is biztosítunk internetelérést.

## Szerver szolgáltatások
- dhcp az állomások IP-konfigurációjához
- konfiguráció tftp-ben való mentése
- fájlok ftp szerveren való tárolása
- belső levelező szerver
- saját webszerver
- dns szerver az ftp-, levelező- és web szerverek eléréséhez


# VLAN felosztás
VLAN number|Name|Szerepe|IP|Portok|Gépek
---|---|---|---|---|---
VLAN10|MANAGEMENT|Rendszergazda|192.168.2.0/25|fa0/1|PC0
VLAN20|VEZETOSEG|A vezetők|192.168.2.128/25|fa0/2<br>fa0/3|PC1<br>PR1
VLAN30|HR|HR osztály|192.168.3.0/25|fa0/4<br>fa0/5|PC2<br>PR2
VLAN40|MARKETING|Marketingesek|192.168.3.128/25|fa0/6|PC3


# Forgalomirányítás
Dinamikus Forgalomirányítási módszer MD5 hitelesített OSPF protokoll használatával.


# A hálózat biztonsága

## Portvédelem alkalmazása
Mivel a cég méretét tekintve nem számítunk sok felcsatlakoztatott eszközre és azok helye korlátozott, szigorú limitációkat vezethetünk be, hogy elkreülhessük az ellenőrizetlen kapcsolatokat.

## Jelszavak
A hálózati eszközöket titkosított jelszó védi, így azokat nem lehet visszaolvasni a konfigurációból.

## Távoli elérés biztosítása
SSH konfigurálása a biztonságos kapcsolat érdekében

## DHCP védelem
DHCP Snooping alkalmazása a DHCP támadások enyhítése érdekében.

## ARP védelem
DAI alkalmazása az ARP támadások enyhítése érdekében.

## STP védelem
BPDU Guard és Root Guard alkalmazása az STP támadások enyhítése érdekében.

## Egyéb védelem
A következő beállításokat végezzük el, hogy minimalizáljuk a potenciális támadások esélyét:
- nem használt portok lekapcsolása és nem használt VLAN-ba helyezése
- trönk portokon natív VLAN állítása
- trönk portok manuális beállítása DTP letiltása

aaa,acl,logging,snmp


# Címzési terv
Hálózat|Kapcsolat formája
---|---
1.1.1.0/30|router-ISP
10.0.0.0/30|router-router
192.168.0.0|vezeték nélküli hozzáférés
10.0.0.4/30|router-router
192.168.1.0/24|szerver hálózat

A szerver, a forgalomirányítók(kivéve az ISP felöli port), a kapcsolók és a nyomtatók statikus címmel rendelkeznek, míg a gépek és telefonok DHCP segítségével kapnak.