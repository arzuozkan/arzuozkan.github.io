# The Great Escape
tryhackme room: https://tryhackme.com/r/room/thegreatescape

## Açık Portların ve Servislerin Keşfi
nmap aracı ile basit bir tarama gerçekleştirildiğinde 22 ssh ve 80 http portlarının açık olduğu tespit edilir. 

`nmap -sS -sV 10.10.228.177 -Pn`

![image](https://github.com/user-attachments/assets/43d117f3-825f-4bc6-97f3-4a62b5419352)

## Web Uygulama Keşfi
giriş yapma, kursları görüntüleme işlemlerinin gerçekleşebildiği ve admin panelinin bulunduğu çevrimiçi sınıfı uygulamasıdır.

![image](https://github.com/user-attachments/assets/5b416b95-0c06-471c-b791-f0d2669bbe8c)

kaynak kodlarını araştırma, dizin taraması ile bilinmeyen dizinleri tespit etme gibi adımları gerçekleştirelim.

![image](https://github.com/user-attachments/assets/7603807d-0447-406a-b997-2f869b3c64c2)

![image](https://github.com/user-attachments/assets/a47a7e29-190f-4b63-9801-acd073e616bc)

![image](https://github.com/user-attachments/assets/77455ab5-b5c8-48d5-83cf-f21511a5f43a)
