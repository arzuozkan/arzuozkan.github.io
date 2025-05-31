---
title: "Detect Attacks with Sysmon - Cyberexam Lab Writeup"
datePublished: Fri Aug 30 2024 09:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cmbc2olhz000e0alaas7kbsbl
slug: detect-attacks-with-sysmon-cyberexam-lab-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1748686165197/4c969aac-541a-41d4-ba35-617bf7319896.png
tags: sysmon, cyberexam

---

## Görev Tanımı

Windows sistemdeki sysmon loglarını inceleyerek gerçekleşen siber saldırıyı ve tehdit aktörünün tespitini yapın. Saldırganın bıraktığı izleri takip ederek saldırının nasıl gerçeleştiğini ve hangi MITRE ATT&CK tekniklerinin kullanıldığı bulunabilmektedir.

## Lab Çözümü

Hedef sisteme erişim sağladıktan sonra, Event Viewer (Olay görüntüleyicisi) aracını açalım. Application and Services &gt; Microsoft &gt; Windows başlığı altında Sysmon loglarını açalım.

### Loglara göre, ilk gerçekleşen saldırının türüne göre MITRE ATT&CK kodu nedir?

Logları incelediğimizde saldırganın RDP portundan giriş denemeleri gerçekleştirdiğini görebiliriz.

### Saldırının başlama zamanı nedir? **(\*/\*\*/20\*\*/\*:\*\*:\*\*)**

Giriş denemelerinin gerçekleşmeye başlaması saldırının başlangıcı olabilir

### Saldırganın IP adresi nedir?

Sysmon loglarında event id 3 gerçekleşen ağ bağlantıları kayıtlarını gösterir. Ağ bağlantısının kaynak IP, hedef IP ve port numarası bilgileri yer alır. Burada saldırganın RDP portunu kullanarak uzaktan erişim sağlamış olduğunu gözlemliyoruz . Bu olay içerisinde saldırganın IP adres bilgisi yer alıyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748685478545/c48c2ac7-1db6-4e91-bdd4-6bab4bdc8431.png align="center")

### cmd.exe ile çalıştırılan ilk komut hangisidir?

Event ID 1 Process creation kayıtlarında sistemde çalıştırılan komutlar yer alır. Yan kısımda bulunan *Filter Current Log* kısmından bu event ID için filtreleme yapabiliriz. cmd.exe processi oluşturulduktan sonra, whoami komutu çalıştırılmaktadır. Event loglarında da detayları görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748685570153/7236d4ae-1a11-4290-86bb-8cbaa23e69d3.png align="center")

Buradan sonra aslında saldırgan ipconfig ile ağ hakkında bilgi toplamaya çalışmış ve sonrasında mimikatz aracını kullandığını görebiliriz. Bu arada da yetkisini yükseltmiş cyber kullanıcısından en yetkili kullanıcı NT AUTHORITY\\SYSTEM kullanıcısına geçmiş.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729109256611/80b637e0-6cdf-4074-9e06-e36399b7d319.png align="center")

### Kurban tarafından indirilen dosya yolu ?

Saldırgan shell aldıktan sonra yetki yükseltmek için kullandığı dosya indirilmiştir.

### Process migration için kullanılan process id değeri ?

sysmon event id 8 process migration işlemini gösterir. Basit bir ifadeyle, bir processin başka bir process üzerinden çalıştırılması olarak tanımlanabilir.

### Saldırganın backdoor olarak kullandığı uygulamanın ismi nedir?

Saldırı akışında bakıldığında bağlantı RDP ile gerçekleşip daha sonrasında farklı bir porttan devam etmektedir. Backdoor çalıştırıldığında yeni network bağlantısını gözlemleyebiliriz.

### Saldırganın oluşturduğu kullanıcının parolası nedir?

Saldırgan upgrade.exe payloadını çalıştırdıktan sonra mimikatz aracını kullanarak yetkilerini yükseltiyor. Sonrasında OfficeUser isimli bir kullanıcıyı oluşturduğunu tespit ediyoruz.

net user komutunu kullanarak kullanıcıyı oluşturuyor ve ardından oluşturlan kullanıcıyı administrators grubuna ekliyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1729110582423/2f3310a7-7f05-4ee7-a2a1-10f5a813149c.png align="center")

### Zamanlanmış görevin ismi nedir?

Bu işlemlerden sonra saldırgan sistemde kalıcılığını sağlamak amaçlı zamanlanmış bir görev oluşturuyor. Backdoor yazılımı her 5 dakikada bir SYSTEM kullanıcısı tarafından çalıştırılmaktadır. /tn parametresi görevin ismini belirtmek için kullanılır.