---
title: "MITRE: Dumping Browsers: T1217, T1555.003, T1176"
datePublished: Thu Nov 14 2024 05:49:50 GMT+0000 (Coordinated Universal Time)
cuid: cm3gw4vc300190amd5miuh9st
slug: mitre-dumping-browsers-t1217-t1555003-t1176
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1731563183125/c34cf080-52d5-47f6-9021-6bba8e62649f.png
tags: firefox, cyberexam, mitre-attack, browser-forensics

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Dumping Browsers: T1217, T1555.003, T1176 lab çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/dumping-browsers/mitre-dumping-browsers-t1217-t1555-003-t1176](https://learn.cyberexam.io/challenges/mitre/dumping-browsers/mitre-dumping-browsers-t1217-t1555-003-t1176)

---

## Konuya Giriş

Bu lab içerisinde MITRE saldırı tekniklerinden

* T1217 - Browser Information Discovery
    
* T1555.003 - Credentials from Password Stores
    
* T1176 - Browser Extension
    

yer verilmiştir.

Tehdit aktörleri eriştikleri sistemdeki tarayıcı tarafından tutulan yer imleri, kayıtlı hesaplar, tarayıcı geçmişi gibi bilgileri kontrol ederek kullanıcı hakkında bilgi toplar ve giriş bilgilerini ele geçirmeyi amaçlar.

## Laboratuvar Çözümü

Sistemde hangi tarayıcının kullanıldığını anlamak için `dpkg -l | grep 'firefox\|chromium\|chrome'` komutu ile yüklenmiş paketler kontrol edilebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725960218270/e78a2126-a89b-4f69-8938-f1849274c514.png align="center")

home dizininde de ls -al komutunu çalıştırdığımızda tarayıcıya ait .mozilla dizinini görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725960394655/3a4b66e5-00c5-4403-a066-06b49459ff51.png align="center")

### Yer işaretleri içerisinde kaç tane URL bulunmaktadır?

Hangi tarayıcının kullanıldığı tespit edildikten sonra, tarayıcı bazlı keşifler yapabiliriz. Firefox için yer imleri bilgisi places.sqlite içerisinde bulunur.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731562004434/215f45d5-487e-4677-b41e-9d8613ea64d7.png align="center")

Bulduğumuz yer imleri dosyası içerisine girelim. sqlite3 aracını kullanıyoruz. Hedef makinede sqlite3 aracı olmadığı için yerel makineye dosyayı gönderelim.

`scp user@attack:/path/to/places.sqlite places.sqlite`

Sonrasında `sqlite3` ile ilgili dosyayı açalım.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725969315562/51f0e021-558c-4754-9dbd-808dcf2018ee.png align="center")

Burada ilgili tablolar olarak `moz_bookmarks` ve `moz_places` bulunuyor.

### Tarayıcıdan parolayı al. Yer imlerine eklenmiş URL nedir?

Yerel makinede saklanılan kullanıcı bilgilerini otomatik olarak tespit eden araçlar geliştirilmiştir. Bu lab içerisinde [LaZagne](https://github.com/AlessandroZ/LaZagne) aracının `browsers` modülünü kullanacağız.

scp komutu ile yerel makinedeki lazagne dosyasını (masaüstünde v2.4.5.tar.gz dosyası) hedef makineye kopyalıyorum.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725966892926/3421dff2-9498-4e6a-86f8-4a5f6a5bd83e.png align="center")

/tmp dizini içerisine kopyaladıktan sonra arşivden çıkarıp aracı çalıştırıyorum.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731562420099/6eef2044-ccdb-4b53-b013-52279e046ef4.png align="center")

### Saldırı makinesine bağlanın. Yer imlerini içeren .sqlite dosyasının dosya dizini nedir?

İlk soruda kullanılan find komutu ile places.sqlite dizinini `find / -path "*.mozilla/firefox/*/places.sqlite" 2>/dev/null` komutunu kullanarak bulmuştuk.

### Uzantılar nerede?

Uzantıları içeren dizin tarayıcının varsayılan profili altında yer alır.

### Yer imlenmiş URL'e ait kullanıcı adı ve parola nedir?

LaZagne aracının çıktısında kullanıcı bilgilerini tespit etmiştik. Çözüm için `username:password` formatında yazabiliriz.

### Kaç tane uzantı yüklüdür?

Uzantıları içeren dizin içerisinde xpi dosya formatında yüklü uzantılar yer alır. `ls` komutu ile listeleyerek yüklü uzantıların sayısını tespit edebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731562973456/f7ebc83e-e5c4-45cf-b052-273a1ffa696d.png align="center")