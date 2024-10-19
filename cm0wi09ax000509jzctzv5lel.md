---
title: "MITRE: Defense Evasion: T1070.002"
datePublished: Tue Sep 10 2024 13:59:32 GMT+0000 (Coordinated Universal Time)
cuid: cm0wi09ax000509jzctzv5lel
slug: mitre-defense-evasion-t1070002
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725976663465/9035f297-f233-49fb-9fd2-7114ccbdbf4d.png
tags: cyberexam, mitre-attack, defense-evasion

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Defense Evasion: T1070.002 lab çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1070-002](https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1070-002)

---

## Konuya Giriş

Tehdit aktörleri, sistemde bıraktıkları izleri, zararlı aktiviteleri ortadan kaldırmak için sistemde tutulan kayıtları silmektedir. Linux sistemlerde bu kayıtlar /var/log dizini altında tutulmaktadır.

* /var/log/messages : genel ve sistem ile ilgili mesajlar
    
* /var/log/secure - /var/log/auth.log : kimlik doğrulama kayıtları
    
* /var/log/utmp - /var/log/wtmp : giriş kayıtları
    
* /var/log/kern.log : kernel (çekirdek) kayıtları
    
* /var/log/cron.log : planlanmış görevlerin kayıtları
    
* /var/log/maillog: eposta sunucu kayıtları
    
* /var/log/httpd/ - /var/log/apache/ : web sunucu erişim ve hata kayıtları
    
* /var/log/audit/audit.log : sistemde önemli olayların denetim kayıtları
    

## Laboratuvar Çözümü

Sistemde gerçekleşen kullanıcı hareketlerinin kaydını tutan /var/log/audit denetim loglarını inceleyelim

### Logları temizlemek için kullanılan syscall (sistem çağrısının numarası nedir?

Denetim kayıtlarına erişmek için admin kullanıcısından yetkili root kullanıcısına geçmemiz gerekiyor. Sonrasında `cat /var/log/audit/audit.log | grep “delete\|log”` komutu ile kayıtları filtreleyerek görüntüleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725975431139/36b9b26a-4b28-453b-9a66-5bae6e5a94f0.png align="center")

### Tespit makinesine admin kullanıcısı ile giriş yapın. Logları inceleyin. Logları silen kullanıcının UID’si nedir?

audit.log içerisinden ilgili kaydı bulduktan sonra auid, uid gibi bilgiler de gösterilir. auid( audit user id) oturumu başlatan kullanıcıyı temsil eder. Sonrasında bu kullanıcı root olarak yetkisini yükseltip silme işlemini gerçekleştirir.