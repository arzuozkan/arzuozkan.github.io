---
title: "Someone Phished Me! - Cyberexam Lab Writeup"
datePublished: Thu Sep 26 2024 19:11:35 GMT+0000 (Coordinated Universal Time)
cuid: cm1jo767y000408lbeqpx2f3r
slug: someone-phished-me-cyberexam-writeup
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1727200357991/bfe8e368-83a3-4ddb-a455-f6753d5686e9.png
tags: windows, phishing, blueteam, dfir, cyberexam

---

Lab bağlantı adresi: [https://learn.cyberexam.io/challenges/blue-team/incident-response/someone-phished-me](https://learn.cyberexam.io/challenges/blue-team/incident-response/someone-phished-me)

---

## Görev Tanımı

Oliver kullandığı Windows makinesinde, banka hesabının çalındığını farkediyor. Yetkililer ile görüşüyor ve makinesi izole ediliyor. Görevimiz, saldırının nasıl gerçekleştiğini tespit edip analizini yapmak. Hedef makineye erişim için `sudo xfreerdp /v:ip /u:Oliver /p:'Nk150Parker2024*!' /dynamic-resolution /cert:ignore +clipboard /f` komutunu kullanabiliriz.

## Laboratuvar Çözümü

Hedef sisteme bağlandığımız zaman Oliver’ın kullandığı eposta uygulaması Thunderbird uygulamasını açalım. Gelen epostaları inceleyelim. Silinen epostalar da mevcut oradaki epostalar da incelenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727201322125/3a8047cc-8b64-45f8-a445-4d4240b5b49c.png align="center")

Daha detaylı bilgilere erişmek için sağ üst köşede yer alan **More** seçeneğine tıklayalım ve **View Source** kısmına gidelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727201460426/a41d657b-6606-4bdb-88b4-ddae705f5007.png align="center")

Epostanın gönderici, alıcı, hangi uygulama ile gönderilmiş içeriği gibi bilgiler yer almaktadır. Eposta arayüzünde, gönderen ve içerikler yanıltıcı olabilir. Bu yüzden eposta kaynağını incelemek gerekebiliyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727202455706/e77e7014-acad-41a8-ba44-3c73ad825a81.png align="center")

Eposta kaynağı içerisinde, gönderilen epostanın kullanıcıya gelene kadar iletildiği sunucular da yer almaktadır. Bunu daha detaylı görebilmek için epostayı .eml dosya formatında kaydedelim. PhishTool gibi analiz araçlarını kullanmak faydalı olacaktır. PhishTool aracı çevrimiçi bir platform olup bir kullanıcı hesabı açmayı gerektiriyor. Bu işlemleri gerçekleştirdikten sonra üst kısımda yer alan **Analysis &gt; Choose file** ile kaydettiğimiz eposta dosyasını yükleyelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727203172322/e669d7d0-bcc8-4727-acae-f8b982d8cb69.png align="center")

### Zararlı dosya hangi eposta adresinden gelmiştir?

Karşımıza görseldeki gibi bir arayüz çıkmaktadır. **Headers** bölümünde epostanın başlık bilgileri yer almaktadır. **From** bilgisinde gönderenin eposta adresi bulunuyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733922707849/65d5b7e2-7664-4761-8f0c-b1d63c902548.png align="center")

Received lines kısmında epostanın kullanıcıya ulaşana kadar iletildiği sunucular yer almaktadır. Toplamda 6 farklı sunucu üzerinden gelerek iletilmiş olduğunu gözlemleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727203615630/1e76059d-0d8f-4270-8dee-63105444bcd8.png align="center")

### Sistemi hacklemek için kullanılan zafiyetin CVE kodu nedir?

Epostada ek olarak gönderilen arşiv dosyasını indirip içeriğini açalım. Ancak makine içerisinde **Downloads** klasöründe bu dosya var. Dosya türünün doğruluğunu test edelim. Dosyaya sağ tıklayıp **Properties** seçeneğinden dosya tipinin aslında pdf değil bir cmd script dosyası olduğunu görüyoruz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733922769732/fc544a3a-ddf6-4d8a-a9ba-e7ddb04965ed.png align="center")

Dosyanın içeriğini notepad veya bir text editörü yardımı ile açtığımız zaman (Sağ tıklayıp Edit seçeneğini işaretleyelim) encode edilmiş bir metin ve powershell komutu görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727205971223/a9ecf8f6-2e36-42ef-94ad-b603020b36a2.png align="center")

Okunabilir hale getirmek için (bu işlem deobfuscation olarak isimlendirilebilir) çeşitli araçlar bulunur. CyberChef aracı bunlardan bir tanesi. Burada sadece Base64 ile decode edilmesi yeterli olmayabilir. Okunabilir olması için Decode text formatı da UTF-16LE olarak Powershelle uygun bir şekilde ayarlanmalıdır. Bu yöntem için [**Malicious Powershell Deobfuscation Using CyberChef**](https://medium.com/mii-cybersec/malicious-powershell-deobfuscation-using-cyberchef-dfb9faff29f) yazısını inceleyebilirsiniz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727206811191/d675bc9f-1ca9-4ae3-9a08-91e292e9eec8.png align="center")

**Zafiyeti araştırma aşaması detayları**

Saldırganın sistemi hacklemek için kullandığı zafiyetin tespitinde windows event loglarını inceleyerek bulunabileceğini düşünerek, şüpheli event ID 4624(başarılı giriş), 4625(başarısız giriş), 4672(yeni oturum açan kullanıcıya özel yetki atanması) gibi logları oluşturabilecek ilgili zafiyetleri araştırmıştım. ChatGPT’ye logları yorumlatarak olası zafiyetleri tahmin etmesini istemiştim. Linux makinesinden nmap ile tarama gerçekleştirdim dışarıya açık herhangi bir zafiyetli servisin kullanımını kontrol ettim rdp veya samba protokolleri ile ilgili zafiyetlere baktım yine bir sonuca ulaşamadım.

Uzun bir araştırma sonucunda ilgili zafiyet WinRAR uzaktan kod çalıştırma (RCE) olarak karşımıza çıkıyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727267828127/7a98bf3c-2dd4-4156-9dc4-579be1f99ee7.png align="center")

Zafiyet WinRAR’ın yüklü olan versiyonunda bulunmaktadır. Windows masaüstünde bulunan WinRAR uygulamasını açıp versiyonunu kontrol edelim **Help &gt; About WinRAR…** seçeneğine tıkladığımız zaman uygulama hakkında bilgilere erişebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733922871614/faa695bf-e33a-4bcb-8c53-763b09a42238.png align="center")

Arşiv içerisinde de aynı isimde pdf ve klasör bulunmaktadır. Klasör içerisinde de zararlı bat dosyası bulunuyor. Bu zafiyet sayesinde kullanıcı farketmeden pdf dosyası ile birlikte zararlı dosyayı da çalıştırmaktadır. Saldırgan klasör ve dosya isimlerini aynı olacak şekilde, arşiv dosyasını oluştururken düzenlemiş.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727269051900/6db57488-482f-46ac-a16a-53bab704bd76.png align="center")

Kullanıcı pdf dosyasını açtığı zaman WinRAR processin içinde cmd.exe processi altında powershell.exe ve conhost.exe processlerinin başlatıldığını görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727269633225/1cc85231-44d7-4d51-be94-7f2acf6c88d8.png align="center")

Bu işlem sonlandıktan sonra powershell.exe processi tekrardan başlatılıyor. Process detaylarında çalıştırılan komut ve yapılan TCP/IP bağlantıları görüntülendiğinde zararlının çalıştığını gözlemlemiş oluyoruz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727269590447/3f47441a-4297-4167-8158-7d6fd9f8d287.png align="center")

### Saldırganın IP adresi nedir?

Saldırganın IP adresini çözülen Powershell kodu içerisinde tespit etmiştik. Bu kod reverse shell ile hedef makine arasında bağlantıyı sağlamak amaçlı kullanılıyor. Saldırgan, kod içerisinde verdiği IP adresine karşı 4443 portunda bir TCP bağlantısı oluşturmayı hedeflemektedir.

### Saldırganın sisteme eriştikten sonra kurduğu zararlı yazılımın ismi nedir?

Kurulan zararlı, **C:\\Users\\Oliver\\Documents** dizini altında yer almaktadır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733923011919/24cd02e5-2d46-4da7-ab24-9b19106f048b.png align="center")

### Saldırgan ne zaman log silme işlemini gerçekleştirmiştir?

Event viewer (Olay görüntüleyicisi) aracı ile uygulama, sistem ve güvenlik gibi gerçekleşen olayların kayıtlarına ulaşabiliriz. Her bir olay(event) ID değerine sahiptir. Saldırgan zararlı aktivitelerinin izlerini temizlemek için logları silebilir. Bu işlem sonucunda Windows event log kaynaklı sistem loglarında **event ID 104** numaralı kayıt oluşur. Güvenlik kayıtlarının silinmesi ise security event ID 1102 olayını tetikler. Sağ kısımda yer alan **Filter Current Log** seçeneğinden ilgili event ID’ye göre filtrelemeyi gerçekleştirelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727376119493/b5ee7b4f-2018-497a-bd7e-b50cf737ad44.png align="center")

Filtreyi uyguladıktan sonra bir tane event listelendiğini görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733923099580/c7b6ef82-5423-4946-9e8f-3ecef6094dfb.png align="center")

Diğer bir silme işlemi de güvenlik loglarında gerçekleşmiştir. Bunun içinde 1102 event ID’ye göre filtreleme yaparak erişebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733923131635/d6cd5d66-1574-4447-b173-8c6c7656107d.png align="center")

### Saldırganın çaldığı parola bilgisi?

Powershell komut geçmişi görüntülendiğinde saldırganın çalıştırmış olduğu komutları görebiliriz. Komut geçmişi `C:\Users\<Kullanıcı>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\` dizininde **ConsoleHost\_history** dosyasında tutulur. `Get-History` komutu geçmişi görüntülemek için kullanılabilir ancak sadece açılan powershell oturumundaki komutları saklar, powershell kapatılıp açıldığı zaman silinir. Ek olarak komut geçmiş dosyasının nerede tutulduğunu `Get-PSReadLineOption` komutunu kullanarak öğrenebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1727213219306/bcce0dea-5e19-4e8e-ad74-9e911ce525a9.png align="center")

Yukarıdaki komutu kullanarak saldırgan, sistemdeki parola bilgisi içeren xml, txt ve ini dosyalarını araştırmıştır. Aynı komutu çalıştırdığımız zaman, **C:\\Users\\OliverMusic\\Bank.txt** içerisinde parola bilgisine ulaşabiliriz. Bu dosya içerisinde Oliver kullanıcısına ait banka hesabı kimlik bilgileri bulunuyor.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733923175199/eaa0c3c7-77e4-4edd-9bda-33dbf65afe54.png align="center")

### Saldırının MITRE ATT&CK Teknikleri

* T1566.001 Phishing Spearphishing Attachment (Initial access/ İlk erişim)
    
* T1203 Exploitation for Client Execution (Execution/ Çalıştırma)
    
* T1105 Ingress Tool Transfer ( Command and Control/ Komuta Kontrol)
    
* T1071.001 Application Layer Protocol: Web Protocols (Command and Control, Execution/ Komuta Kontrol, Çalıştırma)
    
* T1140 Deobfuscate/Decode Files or Infromation (Execution/ Çalıştırma)
    
* T1081 Credentials in Files (Credential access/ Kimlik bilgilerine erişim)
    
* T1070.001 Indicator Removal on Host: Clear Windows Event Logs (Defense evasion/ Savunmayı atlatma)