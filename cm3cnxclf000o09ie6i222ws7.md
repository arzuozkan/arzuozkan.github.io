---
title: "MITRE: Defense Evasion: T1574.006"
datePublished: Mon Nov 11 2024 06:48:58 GMT+0000 (Coordinated Universal Time)
cuid: cm3cnxclf000o09ie6i222ws7
slug: mitre-defense-evasion-t1574006
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1731307680916/6fd0cb64-9982-47c3-b499-efd74f466517.png
tags: privilege-escalation, cyberexam, mitre-attack, defense-evasion

---

Lab linki: [https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1574-006](https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1574-006)

---

## Görev Tanımı

T1574.006 MITRE tekniği, tehdit aktörünün sistemde kalıcılık (persistence), yetki yükseltme (privilege escalation) veya savunma atlatma (defense evasion) gibi amaçlarla bir uygulamanın yürütme akışını, dinamik bağlayıcı (dynamic linker) ortam değişkenini manipüle ederek değiştirmesidir. Dinamik bağlayıcı (dynamic linker), uygulamanın çalıştırılma zamanında dinamik olarak yüklenecek paylaşımlı kütüphaneleri yükler. Saldırgan, kullanılan kütüphaneyi kendi oluşturduğu aynı isimdeki zararlı payload ile değiştirebilir.

Linux sistemlerde, kütüphane bilgisi ve dizin yolu `LD_PRELOAD` ortam değişkeninde veya `/etc/ld.so.preload` dosya içeiğinde tanımlanır.

## Lab Çözümü

### Tespit makinesine bağlanın. Zararlı kütüphane nerededir?

İlk olarak varsa çalıştırılan komut geçmişi `.bash_history` dosyası kontrol edilebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731304731536/0dde6a77-56f6-4cf8-b70b-6de70e908ce8.png align="center")

Çalıştırılan komutlara baktığımızda kullanıcı `lib.c` isminde bir dosya oluşturmuş ve onu gcc derleyicisini kullanarak zararlı kütüphane oluşturmaktadır. İşlemini bitirdiğinde de `audit.log` kayıtlarının silindiğini de görebiliyoruz.

### Zararlı kütüphaneyi yüklemek için hangi yapılandırma değiştirilmiştir?

Komut geçmişinde hangi dosyada değişiklik yapıldığı görünmektedir. `/var/log/auth.log` kayıtları da incelenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731305439949/f6050db6-e439-404c-b41e-3947db163088.png align="center")

Dosyanın içeriğinde `LD_PRELOAD` değişkeni tanımlanmıştır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731305522290/6233725b-0fd2-49b7-8315-ea0afe35a60e.png align="center")

### Reverse shell bağlantısını dinleyen saldırganın IP adresi nedir? Port?

Oluşturulan zararlı kütüphanenin davranışlarını ve sistem çağrılarını incelemek için `strace` komutunu kullanabiliriz. Zararlı kütüphanenin sistem çağrılarında ağ bağlantılarında kullanılan socket, connect gibi çağrılar yer alır. Not olarak, `strace` verilen uygulamayı veya processi sistemde çalıştırır ve eş zamanlı olarak sistemdeki hareketlerini görüntüler.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731307216141/f2f1cf32-7ae7-4645-b4d8-932d7ff43d66.png align="center")