---
title: "MITRE: Credential Access: T1552.004"
datePublished: Mon Sep 09 2024 08:27:35 GMT+0000 (Coordinated Universal Time)
cuid: cm0uqpim9000i0ak5119v8li8
slug: mitre-credential-access-t1552004
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725865818202/ff0f8104-6ec3-438a-aa71-944f3d5ef696.png
tags: credential-access, cyberexam, mitre-attack

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Credential Access: T1552.004 labaratuvar çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/credential-access/mitre-credential-access-t1552-004](https://learn.cyberexam.io/challenges/mitre/credential-access/mitre-credential-access-t1552-004)

## Konuya Giriş

Tehdit aktörleri, erişimi olan sistemde private key olarak isimlendirilen gizli anahtar bilgilerine, sertifika dosyalarına veya kullanıcı giriş bilgilerini içeren dosyalara erişmeye çalışmaktadır. Gizli kriptografik anahtarlar ve sertifika dosyaları kimlik doğrulama, şifreleme ve dijital imza gibi işlemler için kullanılmaktadır. Yaygın olarak bilinen bazı dosya uzantıları arasında, .key, .pgp, .gpg, .ppk, .pem, .cer, .asc yer almaktadır. Bu dosyaların yanı sıra varsayılan olarak kullanılan örneğin ~/.ssh gibi dizinlerde hedeflenmektedir. Koruma amaçlı hassas verileri içeren anahtar veya sertifika dosyaları parola korumalı olabilir ancak tehdit aktörü bu önlemi atlatmak için kaba-kuvvet saldırısı gibi saldırılar gerçekleştirerek parola korumasını ortadan kaldırabilir.

## Laboratuvar Çözümü

### victim kullanıcısı ile saldırı makinesine giriş yapalım. Yetkili kullanıcı ile bağlantıyı sağlayan gizli anahtarın dizini nedir?

`ssh -i victim_key victim@attack` komutu ile sisteme giriş yapalım. `ls -al` dediğimiz zaman .ssh dizinini görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725866823495/0d1642aa-7a69-44a0-b49a-d5f1fab5191a.png align="center")

Dizin içine girdiğimiz zaman authorized\_keys dosyasına erişebiliyoruz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725866906244/2b32a7f7-6840-455c-814b-8ebda131558b.png align="center")

Bu dosya SSH bağlantısını parolasız, anahtar tabanlı kimlik doğrulamayı sağlamaktadır. Public anahtardır. Ancak bu istediğimiz bir bilgi değil. victim dışında admin kullanıcısının home dizinine gidelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725867620830/b798acd5-b6ee-4561-b2e9-4e23a3e63cc8.png align="center")

.ssh içerisine girdiğimiz zaman hem açık anahtar (public key) hem de gizli anahtarın bırakılmış olduğunu görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725867751353/75aecdc5-dd82-451b-8047-d40520ecb0a6.png align="center")

### Bir önceki kısımda bulunan gizli anahtar ile giriş yapalım. Flag değeri nedir?

scp aracı ile gizli anahtarı yerel makineye kopyalıyorum.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725868698006/b612eef0-d7e1-4a63-8afb-2277c8045539.png align="center")

bu gizli anahtarı kullanarak ssh ile admin kullanıcısında oturum açabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725868910611/fd28ae14-285a-4fbc-a3e1-c74de12b17a3.png align="center")

flag dosyası /home/admin dizininde yer alıyordu. Admin kullanıcısının bu dosyaya erişimi bulunmaktadır. Bu sayede flag değerine ulaşmış oluyoruz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725869026697/83e0750d-15be-4892-9833-5d47f8e756ea.png align="center")

Saldırının tespiti için /var/log/auth.log dosyası içerisinde ssh giriş logları yer almaktadır (aşağıdaki örnek loglar chatgpt ile oluşturulmuştur). Bu kayıtlar kontrol edilebilir.

`Sep 9 10:00:01 server sshd[12345]: Accepted publickey for username from 192.168.1.100 port 54321 ssh2: RSA SHA256:ABC...`

`Sep 9 10:00:01 server sshd[12345]: pam_unix(sshd:session): session opened for user username by (uid=0)`

Diğer bir tespit yönteminde ise, komut geçmişine bakılarak, tehdit aktörünün gizli anahtarın dizinine gittiği gözlemlenebilir.