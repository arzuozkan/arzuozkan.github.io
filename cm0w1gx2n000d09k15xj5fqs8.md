---
title: "MITRE: Persistence: T1136.001"
datePublished: Tue Sep 10 2024 06:16:36 GMT+0000 (Coordinated Universal Time)
cuid: cm0w1gx2n000d09k15xj5fqs8
slug: mitre-persistence-t1136001
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725949680554/697fce84-953d-42c6-87c2-a89c2a8422ae.png
tags: persistence, cyberexam, mitre-attack

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Persistence: T1136.001 lab çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/persistence/mitre-persistence-t1136-001](https://learn.cyberexam.io/challenges/mitre/persistence/mitre-persistence-t1136-001)

---

## Konuya Giriş

T1136.001 Create Account: Local Account, tehdit aktörlerinin erişidikleri sistemde kalıcılığı sağlamak için uyguladıkları tekniklerden bir tanesidir. Sistemde yeterli yetkilere sahip olan kullanıcı, yerel kullanıcı hesabı oluşturabilir. Linux işletim sisteminde `useradd` komutu, Windows sistemlerde `net user username /ADD` ve macOS için `dscl -create` komutu kullanılmaktadır. Yerel hesaplar sayesinde sisteme SSH, RDP gibi uzaktan bağlantı gerekmeden erişmeyi sağlar.

## Laboratuvar Çözümü

`ssh -i admin_key admin@detect` komutu ile tespit makinesine giriş yapalım.

### Tespit edilen tehdit aktörünün oluşturmuş olduğu kullanıcının UID değeri nedir?

Linux sistemde kimlik doğrulama gibi güvenlik kayıtlarını `/var/log/secure` veya `/var/log/auth.log` dosyalarında bulunabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733039681576/2dea17b9-9e57-4851-af82-ac1539f512a9.png align="center")

`cat /var/log/auth.log | grep useradd` komutunu çalıştırarak filtreleme yapabiliriz.

Oluşan bu kullanıcı root grubunda ve root dizinine erişimi olacak şekilde ayarlanmıştır. `id <user>` komutu ile kullanıcı id, grup id ve grup bilgilerini görebiliriz.

### Tespit edilen oluşturulmuş kullanıcının adı nedir?

Bir önceki soruda, log içerisinde oluşturulan yeni kullanıcının ismini bulabiliriz.

### Ne zaman oluşturulmuştur?

/var/log/auth.log dosya içeriğinde kullanıcı hesabın oluşma zamanı görülmektedir.