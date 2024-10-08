---
title: "Compromised Machine Analysis - Cyberexam Lab Writeup"
datePublished: Mon Sep 16 2024 05:00:30 GMT+0000 (Coordinated Universal Time)
cuid: cm14je5dr005w09l8hcl206i1
slug: compromised-machine-analysis-cyberexam-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1726410418806/c9321f62-6af4-49a0-8512-67488e23d29b.png
tags: windows, blueteam, dfir, cyberexam

---

Lab bağlantı adresi: [https://learn.cyberexam.io/challenges/blue-team/incident-response/compromised-machine-analysis](https://learn.cyberexam.io/challenges/blue-team/incident-response/compromised-machine-analysis)

---

## Görev Tanımı

Marry Windows bir bilgisayar kullanıyor. Bir süre sonra lisansı sona eriyor ve internetten ücretsiz güncelleyebileceği lisans bulup onu yüklüyor. Sonra banka hesabının ele geçirildiğini farkediyor. Mevcut durumda makine izole hale getirilmiş. Bundan sonra,

* Marry için bir analiz yapılması gerekiyor. İzleri takip edip nasıl hacklendiğinin tespit edilmesi gerekiyor.
    
* `sudo xfreerdp /v:HEDEF_IP /u:Administrator /p:'(S7PTD-Pk!?m92buyO4D-*EJyuhAbdlV' /dynamic-resolution /cert:ignore +clipboard /f` komutu ile Marry’nin bilgisayarına uzaktan bağlantı kurulabilmektedir.
    

Eğer hedef IP adresi verilmediyse `ifconfig` veya `ip a` komutu ile yerel IP adres bloğu tespit edilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726403284767/d2f17826-1cc9-4999-b0d3-6f128807e241.png)

Tespit edilen IP bloğu 10.0.149.0/24, gateway ve host IP adresi çıkarılarak nmap taraması ile hedef IP bulunabilmektedir.

`nmap -sn 10.0.149.0/24 --exclude 10.0.149.1, 10.0.149.92`

---

## Labaratuvar Çözümü

*Not: Lab içerisinde sorular sırayla çözülmemiştir.*

Linux makinesi içerisinde Desktop dizininde tools içerisinde yardımcı olacak Windows process izleme, yetki yükseltme gibi araçlar bulunmaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726403849433/326f186d-c266-4425-aa54-0f514908e421.png)

Windows RDP üzerinden bağlantı kurulduğunda kullanabilmek için python ile sunucu oluşturarak dizini ağ üzerinden paylaşıma açılabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726403769615/9a560f75-42c7-4bef-8c3d-7f66a3f1a631.png  )

`sudo xfreerdp /v:HEDEF_IP /u:Administrator /p:'(S7PTD-Pk!?m92buyO4D-*EJyuhAbdlV' /dynamic-resolution /cert:ignore +clipboard /f` komutu ile uzaktan hedef Windows makineye erişim sağlayalım.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726404055632/75df223a-58e9-464f-9bdf-54fbe3b4cf57.png)

Tarayıcıdan veya komut satırını kullanarak, linux makinesinden paylaşılan dizindeki araçları indirebiliriz. http://10.0.149.92:8000/tools adresine gidelim (linux makinesinin IP adresi ne ise ve HTTP sunucu oluştururken hangi port kullanıldıysa o değerler girilmelidir).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726404669269/e43e2a5e-d311-4e90-8f2f-444b141ca696.png)

Araçların ne için kullanıldığını bilebilirsiniz veya bilmeyebilirsiniz. Biraz araştırmadan sonra yapılacak işlemler için hangi araçların yardımcı olabileceği anlaşılmaktadır. Örneğin çalışan processleri izlemek için Process Explorer veya Process Monitor araçları kullanılır. Oturum açıldıktan sonra başlangıç çalışan uygulamalar, zamanlanmış görevler, kayıt defteri ve servisler gibi çeşitli bilgiler <mark>Autoruns</mark> aracı ile tespit edilebilir.

Autoruns çalıştırdıktan sonra **Options &gt; Scan Options** kısmından <mark>Verify code signatures</mark> seçeneği işaretli değilse seçip tekrar tarama yapabiliriz. Bunun sebebi uygulamaya ait dijital imzayı kontrol ederek doğruluğunu sağlamaktadır. **Check VirusTotal.com** seçeneği, dosyaların hashleri virustotal çevrimiçi platformda tarama yapmak için kullanılır ancak izole ve internete kapalı bir ortam olduğu için bu çözümde kullanmadık.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726405785770/da47433f-9cae-4bd0-b204-0518dbc788a4.png)

Tekrar taramayı bitirdikten sonra Task Scheduler altında doğrulanamayan bir exe dosyası bulunmaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726406308728/007714d5-f82f-4573-8870-61baba8effa1.png)

windowsupdate.exe dosyasına ait MD5 hash bilgisine `Get-FileHash -Path "C:\Users\Administrator\AppData\Local\Temp\windowsupdate.exe" -Algorithm MD5` powershell komutunu çalıştırarak ulaşabiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726406431271/50f287ee-5bfa-46a9-a897-8e8cfacc28eb.png)

Processe sağ tıklyıp Jump Entry seçeneğinden Task Scheduler içerisindeki göreve gidilir

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726407355814/a36b2ddb-108c-4359-81d7-9ee7c2956acd.png)

WindowsUpgrade görevi sistem başlangıç zamanında çalışacak durumda ayarlanmıştır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726407431352/2cd7b829-a706-461e-a013-7c734d77ae68.png)

Zamanlanmış görevi çalıştırıp Process Explorer ile izleyebilir veya Autorun üzerinden hedef dosyaya sağ tıklayıp Process Explorer seçeneğini seçebilirsiniz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726407779047/7aa75617-81a0-45a3-a27a-d99d70891297.png)

Sonrasında çalışan process ile ilgili detaylı bilgi için Properties içerisinden özellikleri görüntülenebilir. Çalışan processe ait dizin bilgisi, çalıştırdığı komut satırı, otomatik başlama konumu, parent processi, çalışmaya başladığı zaman, kaynak kullanım bilgileri, içerdiği metin (strings) ve ağ bağlantıları gibi birçok veriyi göstermektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726407913887/95ee268f-735c-45a6-98a0-6e642796577f.png)

TCP/IP sekmesi içerisinde yapılan bağlantılar listelenir ve Windows makinesinden saldırganın sunucusuna SYN paketi gönderilmeye çalışılmaktadır. Burada, remote address saldırganın veya komuta kontrol sunucusunun IP adresi olabilir.

Zararlının dizinine gidip dosya özellikleri incelendiğinde (sağ tıklayıp Properties seçeneği seçilir) Security, güvenlik başlığı altında dosya sahipleri ve grupların izinleri gösterilmektedir. Bu kısımda 2 benzer isimli kullanıcı adı bulunduğunu görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726408387926/6ccbcab6-d351-41a0-9033-dc0501bf6dd7.png)

*Not: Eğer zararlı görünmüyorsa sistem ayarlarından gizli dosyaları göster (Show hidden files) seçeneğinin aktif olması gerekmektedir.*

`net users` komutu ile sistemde var olan kullanıcılar listelenmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726408531811/d060fff7-32d2-4576-a128-85388f7e9e17.png)

Saldırganın oluşturmuş olduğu kullanıcı burada olabilir. Event Viewer aracı ile de güvenlik kayıtlarından tespit edebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726408645466/2a64c887-40b0-485e-97e2-d594910a0c76.png)

Kullanıcı oluşturma event ID numarası 4720 olarak bilinmektedir. Sağ tarafta **Filter Current Log** seçeneğine tıklayarak filtreleme yapabiliriz. Burada ID bazlı bir filtreleme yapılmaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726408767512/f1651d2e-8674-448a-a236-71c333e7a27b.png)

Yapılan işlemi uyguladıktan sonra kayıtları incelediğimizde oluşturulan kullanıcı tespit edilebilmektedir. Saldırganın kullanıcıyı ne zaman oluşturduğu bilgisi de burada yer almaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726408909092/bf5bf412-7f85-4921-9efa-54dfdb674b82.png)

Powershell üzerinden `Get-WinEvent -LogName Security -FilterXPath "*[System[(EventID=4720)]]" | Format-List TimeCreated, Message` komutu ile de 4720 event id logları görüntülenebilmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726412760971/4804a1d4-b7bf-4c6d-876c-97261198bbfe.png)

Bu zararlının nereden indirildiğini bulmak için tarayıcı geçmişi kontrol edilebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726411455996/1581d932-0ac8-46a1-8487-747935375af2.png)

Windows lisansını ücretsiz kullanmak için zararlının indirildiği web sayfası tarayıcı geçmişinde yer almaktadır. Başka bir yöntemde ise tarayıcı yerel dosya dizininde (örneğin, `C:\Users\Administrator\AppData\Roaming/Mozilla/Firefox/Profile/<PROFILE.default-release\`) places.sqlite dosyasında da tarayıcı geçmişi kaydedilmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726411437013/52acd8f4-2354-4e55-989e-f9d49b8bb0a6.png)

cyberbank.com adresinden yanlış yönlendirme host içerisinden yapılabilir. host dosya dizini Windows sistemlerde `C:\Windows\System32\drivers\etc`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1726409278527/321e094a-3dbf-424c-9a42-80ecd1f650fe.png)

---

## Özet

Marry, windowsfreekey\[.\]com sitesinden ücretsiz lisans exe dosyası indirmiştir. Bu zararlı C:\\Users\\Administrator\\Local\\Temp dizini içerisinde kendini gizlemiş ve sistem startup (başlangıç) programı olarak çalışacak zamanlanmış görev oluşturmuştur. Zararlı dosya çalıştırıldığında, c2 sunucusu ile iletişime geçmeye çalıştığı process TCP/IP bağlantılarından gözlemlenmiştir. Aynı zamanda, host içerisinde kullanıcının bankası cyberbank sayfasını da başka bir zararlı adrese yönlendirecek şekilde değişiklikler yapmıştır. Sistemde Administrator kullanıcısına benzer bir isimle başka bir kullanıcı oluşturmuştur. Bu da Persistence (Kalıcılık) tekniklerinden bir tanesidir.

MITRE ATT&CK teknikleri

* T1566 Phishing (Initial Access/İlk Erişim)
    
* T1204 User Execution ( Execution/Çalıştırma)
    
* T1053 Scheduled task/Job (Persistence/Kalıcılık)
    
* T1547 Boot or Logon Autostart Execution (Persistence/Kalıcılık)
    
* T1136.001 Create Account: Local Account (Persistence/ Kalıcılık)
    
* T1564.001 Hide Artifacts: Hidden Files and Directories (Defense Evasion/ Savunmayı Atlatma)
    
* T1105 Ingress Tool Transfer (Command and Control/ Komuta ve Kontrol)
    
* T1565.001 - Data Manipulation: Stored Data Manipulation (Impact/ Etki)
