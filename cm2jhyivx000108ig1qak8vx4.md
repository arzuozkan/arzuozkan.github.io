---
title: "MITRE: Defense Evasion: T1548.003"
datePublished: Mon Oct 21 2024 20:56:36 GMT+0000 (Coordinated Universal Time)
cuid: cm2jhyivx000108ig1qak8vx4
slug: mitre-defense-evasion-t1548003
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729539480063/0c63ed8b-e17b-489a-93c1-b9ff97fd6cc5.png
tags: cyberexam, mitre-attack, defense-evasion

---

Lab bağlantı adresi: [https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1548-003](https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1548-003)

---

## Görev Tanımı

Savunma atlatma tekniklerinden olan T1548.003, sudo yetki yükseltme mekanizmasının kötüye kullanılmasıdır. Saldırganlar, eriştikleri sistemdeki yetkisini yükseltmek için sudoers dosyasını değiştirme veya sudo caching adı verilen kullanıcının sudo ile yetkisini yükselttikten sonra belirlenen süre içerisinde yüksek yetkileri kullanmaya devam etme durumundan yararlanması ile gerçekleşir. `sudo` komutu sayesinde çalıştırılan komut yüksek yetkiler ile çalışır.

Ayarları içerisinde timestamp\_timeout değeri bulunur ve bu değer ne kadar süre sonrasında sudo ile komut çalıştırıldığında kullanıcı parolası isteneceğini belirler. /var/db/sudo dosyasını oluşturur. Ayrıca her terminal oturumu (tty) için bu değer farklıdır, bir oturum diğerini etkilemez ve birbirlerinden izole durumdadır. Eğer tty\_tickets devre dışıysa, tüm oturumlar aynı sudo cache bilgilerini kullanır.

/etc/sudoers dosyası, kullancıların kullanabileceği komut ve terminallerin tanımlandığı dosyadır. Dosyayı düzenlemek için yüksek yetkilere sahip olunması gerekiyor.

Tespit makineye erişim için `sudo ssh -i admin_key admin@detect` komutunu çalıştıralım. Giriş bilgileri `admin:password`

## Lab Çözümü

### Bu saldırıda düzenlenen dosyanın ismi nedir?

/var/log/auth.log dosyasındaki logları inceleyelim. Saldırgan victim olarak oturum açmış, sed ve visudo komutlarını kullanarak sudo konfigurasyon dosyasının içeriğini değiştirmiştir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729541825803/f4222bd2-0405-42e3-81b7-31e020dcec43.png align="center")

### Dosyanın düzenlenme tarihi nedir?

auth.log dosyası içerisinde düzenlenme tarihi yer almaktadır. Diğer bir yöntem olarak, `stat /path/file` komutu da dosyanın düzenlenme, erişilme ve değiştirilme zamanı ile ilgili bilgileri göstermektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729542410244/f785273c-055d-4f60-b505-1a39c28496b1.png align="center")

### Hangi kullanıcı bu saldırıda düzenleme işlemini gerçekleştirmiştir?

İncelenen log içerisinde victim kullanıcısının sudo yetkilerini kullanarak komut çalıştırdığı görülmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729543526187/ec280700-563b-47ce-84b6-cb801f168a4d.png align="center")

### Yeni oluşturulan timestamp\_timeout değeri nedir?

timestamp\_timeout yeni değeri logların içerisinde de yer alır. Aynı zamanda `sudo -l` komutu ile de görüntüleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729544057775/46b8f212-b5cd-4ec2-843b-f0fe56f7802b.png align="center")