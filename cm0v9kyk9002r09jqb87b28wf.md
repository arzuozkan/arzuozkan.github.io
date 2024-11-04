---
title: "MITRE: Discovery Techniques"
datePublished: Mon Sep 09 2024 17:15:56 GMT+0000 (Coordinated Universal Time)
cuid: cm0v9kyk9002r09jqb87b28wf
slug: mitre-discovery-techniques
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725884180151/7564faf7-6533-49c7-94e8-3ab2a5913150.png
tags: cyberexam, mitre-attack, discovery-techniques

---

Cyberexam platformunda yer alan [**MITRE | ATT&CK öğrenme yolu**](https://learn.cyberexam.io/learning-modules/mitre-att-ck)na ait, MITRE: Discovery Techniques lab çözümünden bahsedeceğim.

Not: Lab bağlantı adresi - [https://learn.cyberexam.io/challenges/mitre/discovery/mitre-discovery-techniques](https://learn.cyberexam.io/challenges/mitre/discovery/mitre-discovery-techniques)

---

## Konuya Giriş

MITRE ATT&CK Keşif teknikleri, tehdit aktörünün eriştiği sistemle ilgili bilgi toplama aşamalarını içermektedir.

* T1619: Cloud Storage Object Discovery
    
* T1614.001: System Location Discovery: System Language Discovery
    
* T1613: Container and Resource Discovery
    
* T1580: Cloud Infrastructure Discovery
    
* T1518.001: Software Discovery: Security Software Discovery
    
* T1497.001: Virtualization/Sandbox Evasion: System Checks
    
* T1201: Password Policy Discovery
    
* T1135: Network Share Discovery
    
* T1124: System Time Discovery
    
* T1083: File and Directory Discovery
    
* T1082: System Information Discovery
    
* T1069: Permission Groups Discovery: Local Groups
    
* T1057: Process Discovery
    
* T1049: System Network Connections Discovery
    
* T1046: Network Service Discovery
    
* T1033: System Owner/User Discovery
    
* T1016: System Network Configuration Discovery
    

## Laboratuvar Çözümü

### Saldırı makinesine ssh ile bağlanın. kullanıcı id değeri nedir?

`id` komutunu kullanarak sistemdeki kullanıcıya ait kullanıcı idsi, kullanıcı adı, grup adı gibi bilgilere erişilmektedir. uid bilgisi, kullanıcının yetkilerini ve erişim kaynaklarını gösterebilir. En yetkili root kullanıcının uid değeri 0' dır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703355373/58d0503a-1865-4509-8139-8c542dd588c4.png align="center")

Saldırganın bu keşif tekniği T1033: System Owner/User Discovery ve T1069: Permission Groups Discovery: Local Groups ile ilikilendirilebilir. Tehdit aktörü, eriştiği sistemdeki bulunan kullanıcıyı, yetkilerini öğrenmek ister. Buna göre saldırı senaryosu gelişmektedir.

### victim kullanıcısının kullanıcı grubu nedir?

Bir önceki soruda kullanılan `id` komutu ile victim kullanıcısının grubu öğrenilebilir. Başka bir yöntemde ise groups victim komutu ile victim kullanıcısının hangi gruplara bağlı olduğu görülebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703388189/c687b30a-8de2-4f3c-bf77-76a75bdcf739.png align="center")

T1069: Permission Groups Discovery: Local Groups tekniği içerisinde yer alır.

### Sistem dili nedir?

Tehdit aktörünün sistem dili keşfindeki amaç, hedefin konumu, bölgesi ve kullanılan dil hakkında bilgi toplayarak saldırılarını özelleştirmek olabilir. Linux sistemlerde, `locale` komutunu kullanarak veya `echo $LANG` komutu ile sistem dili görüntülenebilir. Windows için `Get-WinSystemLocale` powershell komutu kullanılabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703447497/6533e658-f9e1-4646-a991-d61b38e79814.png align="center")

MITRE ATT&CK çerçevesinde ilgili teknik, T1614.001: System Location Discovery: System Language Discovery.

### Sanallaştırılmış bir sistem mi?

Bir sistemin sanallaştırılmış olup olmadığın kontrol etmenin birden fazla yöntemi bulunur. `systemd_detect-virt` komutu bazı Linux sistemlerde sanallaştırma tespiti için kullanılmaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725888690003/3171e49d-a916-4ae3-8ff8-3476adf290ff.png align="center")

Bir başka yöntemde ise sistem kaynaklarını ve mimarisini kontrol etmek olabilir. `lscpu` komutu sistemin CPU mimarisi hakkında bilgi verir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725889214912/324d0aa8-af31-4a23-bc36-14cda5f5fa5a.png align="center")

T1497.001: Virtualization/Sandbox Evasion: System Checks tekniği ile ilgilidir. Kötü amaçlı yazılımlar, sanallaştırılmış, izole edilmiş analiz ortamlarından kaçınmak için bu bilgilere erişmektedir.

### Sistem bir konetynır içerisinde mi?

Bir önceki soru içerisinde systemd-detect-virt komutu çıktısında sistemin docker koneynır olduğunu görmüştük. Başka kullanılan yöntemler arasında .dockerenv dosyasının varlığının tespit edilmesi olabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725889495868/ecf1a8cc-2f37-49e0-a97c-b8b69ede34d7.png align="center")

Bu yöntemlerin dışında, `env` komutu ile ortam değişkenleri kontrol edilebilir, uname -a ile çekirdek ismi ve versiyonu konteynır yapısına özel olabilir (ubuntu 5.15.0-1026-aws).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725890004134/e7d81239-4267-430b-8c43-bc6922d8a136.png align="center")

### Sistemde kurulu güvenlik yazılımları yüklü mü?

T1518.001: Software Discovery: Security Software Discovery MITRE ATT&CK tekniği, sistemde kullanılan güvenlik yazılımlarını kontrol eder. Sisteme karşı gerçekleştirilecek zararlı aktivitelerin tespit edilmemesi, kullanılan güvenlik ürününün türü ve daha önce bulunmuş bir zafiyeti var mı gibi sorulara cevap vermektedir.

Mevcut güvenlik yazılımı kontrolü için birçok yöntem bulunur. Bunlardan birisi, yüklü olan paketleri listelemek olabilir. `dpkg --list | grep -i 'antivirus\|firewall\|filtering\|security'` komutunu kullanarak yüklü olan paketler içerisinde ilgili aracın olup olmadığı kontrol edilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725895490022/caf08fb9-4722-4476-8c2c-feff5f26ab44.png align="center")

ufw (Uncomplicated firewall) varsayılan olarak yüklü olan basit bir güvenlik duvarı aracıdır. Root kullanıcı olmak için admin kullanıcısına geçiş yaptım (victim sudo grubunda değil). `ufw status` komutu ile çalışır durumunu kontrol edelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725895947007/59f3982f-1851-41fc-a002-84df013d5b24.png align="center")

Başka bir kontrol yöntemi çalışan processlere bakmak olacaktır. `ps aux` komutu ile processler listelenir. `grep` ile kelime bazlı filtrelemeler de yapabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725896089512/1f5572f5-15bd-4926-b60a-7431baeb7203.png align="center")

### İzin verilen en fazla kullanıcı giriş deneme sayısı nedir?

Linux sistemlerde parola politikası /etc/login.defs dosyasında bulunabilir. `cat /etc/login.defs` komutu ile dosyanın içeriğini incelediğimizde giriş deneme sayısını görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703526466/00dbaf5f-9894-4478-81c5-104497fcf843.png align="center")

### Parola geçerliliği süresi bitmeden kaç gün önce uyarmaktadır?

Bir önceki soruda kullanılan login.defs dosyasında bu bilgi yer almaktadır. `PASS_WARN_AGE` değeri parolanın geçerliliği dolmadan uyarı vereceği günü belirtmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703569004/5dc1855b-f19b-4951-849d-4fd394f2cde6.png align="center")

T1201: Password Policy Discovery tekniği ile ilişkilendirilebilir.

### PID değeri 1 olan processin adı nedir?

`ps aux` komutu ile veya daha da özelliştirilmiş olarak kullanılan `ps -p 1` komutu hangi processin çalıştığını göstermektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703609625/e9392d8b-ef45-4ab0-86d2-fd40febca98a.png align="center")

MITRE ATT&CK T1057: Process Discovery tekniğidir.

### Sistemde kaç tane ethernet arayüzü bulunur?

ip komutu ile network arayüzleri listelenmektedir. `ip a` veya `ip link show` komutları kullanılabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725897661723/9ee805d0-3e16-4008-9831-b35aa38df81c.png align="center")

Ek olarak, linux sistemlerde `/sys/class/net` dizini altında arayüzlere ait dizinler bulunmaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725897820088/fb942bc3-1dba-4b89-9731-e5d9fea38c73.png align="center")

### Yerel ağ subnet adresi nedir?

ip a komutu arayüzleri ve adreslerini detaylı olarak göstermektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730703674632/fdcd99f0-aab2-4d12-81d0-ab425c3c28b1.png align="center")

### Dış ağın subnet değeri nedir?

`ip a` komutunu kullanarak veya `ip route show` ile görüntüleyerek dış ağ adresini bulabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730704302771/b1c99076-33b3-4ff0-b8bd-cff424b42056.png align="center")