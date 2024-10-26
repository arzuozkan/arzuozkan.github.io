---
title: "MITRE: Execution: T1059.004"
datePublished: Sat Oct 26 2024 22:12:43 GMT+0000 (Coordinated Universal Time)
cuid: cm2qpvo2q000109lb533w0dur
slug: mitre-execution-t1059004
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729446590492/a0f4d29f-fd97-4d52-942c-9167b6a9d229.png
tags: execution, cyberexam, mitre-attack

---

Lab bağlantı adresi: [https://learn.cyberexam.io/challenges/mitre/execution/mitre-execution-t1059-004](https://learn.cyberexam.io/challenges/mitre/execution/mitre-execution-t1059-004)

---

## Görev Tanımı

MITRE Attack T1059.004 tekniği, tehdit aktörlerinin unix shell komutlarını kötü amaçlı kullanmalarını ifade eder. Bu komutlar belirli yetkilerle sistemi her yönüyle yönetebilir. Saldırgan gerçekleştirdiği saldırıda zararlı komutlar çalıştırabilir ve interaktif bir oturuma sahip olabilir. Aynı zamanda sistemde yetki yükseltmek ve kalıcılık için de shell komutları çalıştırılabilir. Sistem denetleme aktif ise audit.log dosyası içerisinde sistemde yapılan çeşitli işlemler ve çalıştırılan komutlar kayıt altına alınır.

Hedef makineye `ssh -i admin_key admin@detect` komutunu çalıştırarak erişim sağlayalım.

## Lab Çözümü

### Saldırganın uid değeri nedir?

/var/log/auth.log dosyası içerisinde sistemde gerçekleşen kimlik doğrulama ve oturum açma işlemlerinin kayıtları yer almaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729449095950/2037ffd3-2b05-4ed3-850a-7c1dfc8a353e.png align="center")

/var/log/audit/audit.log dosyası içerisinde de verilen denetim kurallarına göre kayıtlar tutulmaktadır. Genellikle çalıştırılan komutlar, dosya dizin işlemleri, sistem çağrıları, kimlik doğrulama gibi logları tutar. Buradan da şüpheli çalıştırılan kodları çalıştırırken hangi kullanıcıda oturum açıldığı bilgisini de `auid` değerinde belirtilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729450496811/804ac31b-92ba-4ad2-960a-98d60800d366.png align="center")

### Saldırganın çalışma dizini hangisidir?

/var/log/audit dizini altında yer alan audit.log dosya formatında cwd (current working directory) değeri çalışılan dizini göstermektedir. Saldırgana ait işlemler izlendiği zaman /tmp dizini altında işlemleri yaptığını gözlemleyebiliriz. Log türü CWD (change working directory) bilgisinden öğrenebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729451457070/0a3e211f-b050-4fc8-9912-a1126e8ee529.png align="center")

### Çalıştırılan script nedir?

Saldırgan scripti wget komutu ile indirmektedir. Sonrasında dosyaya çalışma izni verip çalıştırmaktadır. /var/log/audit/audit.log içerisinde incelediğimiz zaman tespit edebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729451991664/03a778ab-1c86-42eb-af72-3730fce7662f.png align="center")

### Scriptin indirildiği IP adresi ve port numarası nedir?

Log içerisinde çalıştırılan komutlar incelenirken wget komutunu görebiliriz. grep aracını kullanarak filtreleme yapıldığında scriptin hangi bağlantıdan indirildiğini bir önceki görselde gözlemleyebiliriz. Kullanıcının IP adresi ve port numarası bağlantıda yer alıyor.

### Saldırının başlama zamanı nedir?

Saldırının başlama zamanı scriptin indirildiği zamandır. audit.log dosyasında zaman formatı saniye cinsinde gösterilmektedir. Bu değeri UTC formatına çevirmek için `date -d @SECOND_VAL` komutunu kullanabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729980153548/104b1f3f-1e2a-4780-887e-2b9376a26207.png align="center")