---
title: "MITRE: Credential Access: T1003.008"
datePublished: Sun Sep 08 2024 22:58:53 GMT+0000 (Coordinated Universal Time)
cuid: cm0u6e58r002b0aid95wogmrx
slug: mitre-credential-access-t1003-008
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730009079454/7e433cec-2aaa-457c-a786-7c71c90f4477.png
tags: credential-access, cyberexam, mitre-attack

---

Cyberexam platformunda yer alan [MITRE | ATT&CK öğrenme yolu](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Credential Access: T1003-008 lab çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/credential-access/mitre-credential-access-t1003-008](https://learn.cyberexam.io/challenges/mitre/credential-access/mitre-credential-access-t1003-008)

---

## Konuya Giriş

Tehdit aktörleri, saldırdığı sistem içerisinde kullanıcıya ait giriş bilgilerine erişebilmektedir. Linux işletim sisteminde bu gibi hassas bilgileri içeren dosyalar arasında `/etc/shadow` ve `/etc/passwd` dosyaları yer almaktadır. Sistemde olan kullanıcılar `/etc/passwd` içerisinde listelenirken, bu kullanıcılara ait parolanın hash bilgisi `/etc/shadow` dosyası içerisinde tutulmaktadır ve varsayılan olarak sadece root tarafından okunabilir durumdadır. `unshadow /etc/passwd /etc/shadow > pass_crack.db` komutu ile shadow içerisindeki parola hashleri çözülmektedir.

## Laboratuvar Çözümü

Kurban makine bilgileri victim:password olarak verilmiştir. `ssh -i victim_key victim@attack` komutu ile ssh bağlantısı yapılabilir. Start Lab butonu ile lab ortamını başlatalım.

![ssh bağlantısı](https://cdn.hashnode.com/res/hashnode/image/upload/v1725827685074/6fcf0f32-3fed-4665-af57-ee109d90e77a.png align="center")

### Sistemde kaç tane kullanıcı bulunmaktadır?

Kaç tane kullanıcı olduğunu bulmak için /etc/passwd dosyasının içine bakabiliriz. UID değeri 1000 ve üzeri olan kullanıcılar insan kullanıcılardır. Diğer kalanlar, sistem kullancısı olarak bilinir.

`awk -F: '($3 >= 1000) && ($7 !~ /nologin/ && $7 !~ /false/) {print $1}' /etc/passwd` komutunu kullanarak sistem kullanıcılarını eleyip normal kullanıcıları listeleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733039133249/21d83576-acdc-4057-961d-f481d818bc73.png align="center")

### Root olmayan kaç kullanıcı vardır?

Root en yetkili kullancıdır. UID değeri 0' dır. Belki burada admin kullanıcısı sudo grubundadır o yüzden onu root olarak sayabilir. `cat /etc/group` veya `getent group` komutu ile grup bilgisi öğrenilebiliyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725828609441/d53e6257-6dba-4202-8dfa-f1126c90068b.png align="center")

### Root olmayan kullanıcıların kullanıcı grubu nedir?

Non-root bir kullanıcı seçelim, örneğin user. Bu kullanıcının grubunu `groups user` komutu ile öğrenebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733039196513/e60faa23-f143-4f7e-afda-7cb93c54774e.png align="center")

### John'un parolası nedir?

Johnun parolası için öncelikle `/etc/shadow` dosyası kontrol edilmeli. Bunun için sudo su ile root yetkili kullanıcısına geçiş yapalım. Kurban makinede, unshadow ve john araçları bulunmuyor. bu yüzden kendi makinemize shadow ve passwd dosyalarını alalım. Bu işlemi scp ile de gerçekleştirebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725835379458/a9dd27b7-e67c-4250-866e-81ce31479267.png align="center")

shadow dosyası permission denied hatası verecektir. Onun da ya izinlerinin değiştirilmesi gerekiyor veyahut içeriği kopyalanabilir. unshadow komutu ile passwd ve shadow içerikleri birleştirilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725835959853/2b56ddd7-b2b0-46b5-bca0-10743ae419ec.png align="center")

Oluşan dosyayı john aracını kullanarak parola hashleri kırılabilir. `john cracked.db` komutu çalıştırılır ve kullanıcıların parolaları elde edilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725836114483/85f3f557-8a36-4951-96b2-c2455d7e2a36.png align="center")

### Adminin parolası Nedir?

admin kullanıcısının parolası john aracını kullanarak bir önceki soruda elde edilmiştir.