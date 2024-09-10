---
title: "MITRE: Persistence: T1136.001"
datePublished: Tue Sep 10 2024 06:16:36 GMT+0000 (Coordinated Universal Time)
cuid: cm0w1gx2n000d09k15xj5fqs8
slug: mitre-persistence-t1136001
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725949680554/697fce84-953d-42c6-87c2-a89c2a8422ae.png
tags: persistence, cyberexam, mitre-attack

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Persistence: T1136.001 labaratuvar çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/persistence/mitre-persistence-t1136-001](https://learn.cyberexam.io/challenges/mitre/persistence/mitre-persistence-t1136-001)

---

## Konuya Giriş

T1136.001 Create Account: Local Account, tehdit aktörlerinin erişidikleri sistemde kalıcılığı sağlamak için uyguladıkları tekniklerden bir tanesidir. Sistemde yeterli yetkilere sahip olan kullanıcı, yerel kullanıcı hesabı oluşturabilir. Linux işletim sisteminde `useradd` komutu, Windows sistemlerde `net user username /ADD` ve macOS için `dscl -create` komutu kullanılmaktadır. Yerel hesaplar sayesinde sisteme SSH, RDP gibi uzaktan bağlantı gerekmeden erişmeyi sağlar.

## Labaratuvar Çözümü

`ssh -i admin_key admin@detect` komutu ile tespit makinesine giriş yapalım.

### Tespit edilen tehdit aktörünün oluşturmuş olduğu kullanıcının UID değeri nedir?

Linux sistemde kimlik doğrulama gibi güvenlik kayıtlarını `/var/log/secure` veya `/var/log/auth.log` dosyalarında bulunabilir.

`cat /var/log/auth.log | grep useradd` komutunu çalıştırarak filtreleme yapabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725948235307/9c7ec95e-68ad-48d1-a186-2c713d6651ec.png align="center")

markus isminde oluşan bu kullanıcı root grubunda ve root dizinine erişimi olacak şekilde ayarlanmıştır. `id markus` komutu ile kullanıcı id, grup id ve grup bilgilerini görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725948853549/8d98cba2-5cbc-46e8-a797-a47c1840e229.png align="center")

### Tespit edilen oluşturulmuş kullanıcının adı nedir?

Bir önceki soruda cevabı bulunmuştur. Oluşan yerel kullanıcı markus.

### Ne zaman oluşturulmuştur?

/var/log/auth.log dosya içeriğinde kullanıcı hesabın oluşma zamanı görülmektedir.