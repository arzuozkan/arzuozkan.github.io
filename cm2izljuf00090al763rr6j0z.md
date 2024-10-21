---
title: "MITRE: Defense Evasion: T1222.002"
datePublished: Mon Oct 21 2024 12:22:38 GMT+0000 (Coordinated Universal Time)
cuid: cm2izljuf00090al763rr6j0z
slug: mitre-defense-evasion-t1222002
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729511407934/2dd6d091-4447-4d43-9f02-83074fc0e126.png
tags: cyberexam, mitre-attack, defense-evasion

---

Lab bağlantı adresi: [https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1222-002](https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1222-002)

---

## Görev Tanımı

T1222.002 linux ve macos dosya ve dizin izinlerinin düzenlenmesi savunma atlatma teknikleri arasında yer alır. Tehdit aktörleri, sistemde düzenlediği saldırıyı gerçekleştirebilmek için gerekli dosya ve dizinlerin izinlerini düzenleyebilir. Linux sistemlerde dosya ve dizin sahiplik ve izinler ACL (access control list) adı verilen yapı ile kontrol edilir.

Bu düzenlemeler, chown (change owner) ve chmod(change mod) komutları tarafından gerçekleştirilir. Saldırının tespiti için komut geçmişi, audit logları incelenir.

## Lab Çözümü

/var/log/audit/audit.log dosyasını incelemeden önce kısaca log formatına göz atalım. İçeriğinde çeşitli türlerde log olduğunu görebiliriz. PROCTITLE, USER\_START, CRED\_ACQ, USER\_AUTH, SYSCALL, PATH, EXECVE, CWD, SOCKADDR bu türlerden bazılarına örnek olarak verebiliriz. Log formatını incelediğimiz zaman

* türü
    
* gerçekleşme zamanı,
    
* pid - process bilgileri
    
* uid,auid - kullanıcı kimlik bilgileri
    
* syscall - sistem çağrıları
    
* comm - çalıştırılan komut
    
* exe - çalıştırılan dosya dizini
    
* cwd - komutun çalıştırıldığı dizin
    

bilgiler bulunmaktadır.

Daha detaylı bilgi için [Linux auditd for Threat Detection \[Part 1\]](https://izyknows.medium.com/linux-auditd-for-threat-detection-d06c8b941505) yazı serisini inceleyebilirsiniz.

Audit logların analizi için grep ile filtreleyebiliriz veya ausearch, aureport gibi araçları kullanabiliriz.

### Tespit makinesine bağlanın. Logları inceleyin ve saldırıyı gerçekleştiren kullanıcının id değeri nedir?

Log içerisinde yer alan `auid` değeri olayı gerçekleştiren kullanıcının kimliğini temsil eder. `grep "EXECVE" audit.log` komutunu kullanarak çalıştırılan komut ve argumanları listeleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729509977877/e1838a91-8dc1-46e3-9b34-7410f0ae1142.png align="center")

Diğer bir yöntemde, aureport aracını kullanarak çalıştırılan komutları listeleyebiliriz. `aureport -i --executable --summary`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729510191119/24272b8c-6130-403e-86ce-8d73c5132181.png align="center")

Bu işlemlerden şüpheli olan komutların SYSCALL logu incelenebilir. Çünkü bu tür loglar daha detaylı veriler içeriyor. Örneğin, chmod komutunu çalıştıran kullanıcıyı bulmak için `grep -B 5 "chmod" /var/log/audit/audit.log` komutunu kullanabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729510945313/839662df-4cd2-4b74-85b3-17c79e057fac.png align="center")

### Saldırıda kullanılan komutun argümanı nedir?

Saldırgan `wget` komutu ile kullanacağı zararlı dosyayı hedef sistemde /tmp dizinine yükleyip, ardından `chmod` komutunu kullanarak çalışma modunu değiştirmiştir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729506301445/37667c9f-ad5b-439c-b84f-7f6ccf31b678.png align="center")

### Saldırıda düzenlenen dosya hangisidir?

Bir önceki soruda chmod komutu ile çalışma modu düzenlenen dosya saldırı için kullanılmıştır.