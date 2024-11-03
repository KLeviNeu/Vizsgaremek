# Etherchannel
A port-channel aktív és minden portja működik
![A port-channel aktív és minden portja működik](image-2.png)  
A keret az Fa0/22,24-es portokon át megy
![A keret az Fa0/22,24-es portokon át megy](image.png)  
Miután ezt a két portot lekapcsoljuk (ezzel szimulálva azok kiesését)![alt text](image-3.png)  
A keret az Fa0/21,23-as portokon át fog menni
![A keret az Fa0/21,23-as portokon át fog menni](image-4.png)
Valamint az is látszik, hogy az STP nem külön tiltja le a portokat, hanem az egyik ágon csak
# STP
![Port-channel 1 tiltva STP által](image-5.png)
Ha mindkét útvonal elérhető az egyiket az STP letiltja amíg a másik elérhető, ezzel elkerülve a hurkok kialakulását.  
Ha azonban az aktív út megszakad, a hálózat a másik utat választja
SZimuláljuk ezt azzal, hogy a port channel 2 portjait letiltjuk
![alt text](image-6.png)  
jól látható hogy ezúttal a fenti úton ment a csomag és mivel nincs is más út, az STP sem tiltotta.
# DHCP
PC0 megkapja a címeket DHCP-n
![PC0 megkapja a címeket DHCP-n](image-7.png)
# Telnet/SSH
PC0 connected to SW0 via Telnet
![PC0 connected to SW0 via Telnet](image-9.png)  
PC2 connected to SW4 via Telnet
![PC2 connected to SW4 via Telnet](image-10.png)
# HSRP
Láthatjuk, hogy R1 az aktív router
![Láthatjuk, hogy R1 az aktív router](image-11.png)
és itt mennek a csomagok
![és itt mennek a csomagok](image-15.png)
De miután kiesik(Lekapcsoljuk a portot)
![aktívvá válik R2](image-12.png)
aktívvá válik R2
![és a csomagok is itt mennek](image-13.png)
és a csomagok is itt mennek
R1 visszaveszi az aktív szerepet
![R1 visszaveszi az aktív szerepet](image-14.png)
# Biztonság
## Portbiztonság
![alt text](image-18.png)
felcsatlakoztatunk egy porta egy nem megbízható eszközt
![alt text](image-17.png)
A kapcsoló eldobja a keretet
![alt text](image-19.png)
És a port sértés számláló is növekedett
## DHCP snooping
Felcsatlakoztatunk egy nem megbízható DHCP szervert
![A kapcsoló dobja a keretet](image-20.png)
A kapcsoló dobja a keretet
# Dinamikus forgalomirányítás

# NAT, Portátirányítás
Jól látható, hogy a router natolta a csomagot mielőtt, kiengedte volna
![alt text](image-21.png)
# WiFi
Látható, hogy az eszközök megkapják az összes szükséges információt
![alt text](image-22.png)
# Szerver szolg.
## WEB
![alt text](image-23.png)
## FTP
![alt text](image-24.png)
![alt text](image-25.png)
## MAIL
![alt text](image-26.png)
## TFTP
![alt text](image-27.png)
# GRE alagút
![alt text](image-28.png)