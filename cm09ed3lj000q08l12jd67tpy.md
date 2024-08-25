---
title: "Splunk for Security 101"
seoTitle: "Splunk Basics for Security"
seoDescription: "Learn Splunk basics for security: commands, searches, statistical analysis, and practical challenges to enhance cybersecurity"
datePublished: Sun Aug 25 2024 09:58:51 GMT+0000 (Coordinated Universal Time)
cuid: cm09ed3lj000q08l12jd67tpy
slug: splunk-for-security-101
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724577633907/493893d6-1038-4783-a7a6-2017e2861fe5.png

---

**Kaynak:** [https://bots.splunk.com/](https://bots.splunk.com/)

Learn &gt; Getting Started with Splunk for Security

---

Genel Bakış

* splunk temel kullanımı
    
* Arayüzü
    
* Arama sorguları ve operatörler
    
* stats, sort ve table komutları ve wildcard kullanımı
    
* head ve tail komutları
    
* rex - bir alan içeisinden değeri çıkarmak
    
* eval ve stats
    
* timechart
    
* transaction
    
* iplocation ve geostats
    

Örnek olarak `index=botsv1` [`imreallynotbatman.com`](http://imreallynotbatman.com) aramasında bulunan boşluklar AND operatorü olarak kullanılır.

Arama komutları pipe ( | ) sembolü ile ayrılır.

Wildcard kullanımı desteklenir, örneğin `sourcetype=stream:*` veya `*.exe` gibi

---

# Checkpoint #1

10 Ağustos 2016 tarihli aktivitelere göre cevaplanmalıdır.

## Soru 1: stream:http kaynak türüne göre [imreallynotbatman.com](http://imreallynotbatman.com) sitesinin zafiyetlerini tarayan IP adresi nedir?

`index="botsv1" sourcetype="stream:http" "`[`imreallynotbatman.com`](http://imreallynotbatman.com)`" | stats count by src_ip`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576610762/6e7281c5-84fb-4455-aa28-3719894b1095.png align="center")

40.80.148.42 ip adresinin aktiviteleri görüntülenir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576600577/9d0e63e7-48eb-4ac9-b553-c47d087b3b46.png align="center")

Arama kısmında SQL sorguları çalıştırdığı gözlemlenmektedir. Kullanılan payload, [`http://imreallynotbatman.com/joomla/index.php/component/search/?searchword=(select(0)from(select(sleep(3)))v)/*'%20(select(0)from(select(sleep(3)))v)%20'&quot;%20(select(0)from(select(sleep(3)))v)%20&quot;*/`](http://imreallynotbatman.com/joomla/index.php/component/search/?searchword=(select(0)from(select(sleep(3)))v)/*'%20(select(0)from(select(sleep(3)))v)%20'%22%20(select(0)from(select(sleep(3)))v)%20%22*/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576592601/f007633e-d622-4783-993d-11b6e2b1204b.png align="center")

## Soru 2: suricata kaynak türüne göre olası taranılan IP adresi nedir?

Bir önceki soruda attacker IP adresini tespit etmiştik. Buna göre

`index="botsv1" sourcetype="suricata" src_ip="40.80.148.42" | stats count by dest_ip | sort -count`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576582228/26819f73-3a02-495b-811b-f25d86d54c7b.png align="center")

veya alert.category içerisinden Web app attack olan dest\_ip adreslerini listeleyerek ulaşabiliriz.

`index="botsv1" sourcetype="suricata" src_ip="40.80.148.42" alert.category="Web Application Attack" | stats count by dest_ip`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576572089/a0719d1f-25ee-4d27-8efe-d5dbd73d1c95.png align="center")

Çıkan sonuçlar arasında da alert signature içerisinde SQLi saldırısı tespit edilmiştir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576563963/6a6dd214-6c64-4448-b1f1-f925026ee2ba.png align="center")

## Soru 3: IIS veri türüne göre, hangi kullanıcı ajanı (user agent) web taramalarında en çok görülmüştür?

Sonucu bulmak için IIS kaynak türü seçilir ve cs\_User\_Agent filed incelenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576538312/c6fb5669-8c3e-4bbb-9b55-41d92eb3ef59.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">stats: verilen alan(field) değerine göre verileri gruplandırır ve istatistiksel işlemleri gerçekleştirebilir.</div>
</div>

## Challenge Question #1 : [imreallynotbatman.com](http://imreallynotbatman.com)'daki tarama sırasında döndürülen ilk 10 URL hangileridir?

url detaylara bakıldığında Top 10 değerler listelenir

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576514135/34006c3b-ee30-4f9e-9b06-65a7875c06c3.png align="center")

aynı şekilde arama sorgusunda da stats ve sort komutu ile de bu sonuç elde edilebilir.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `src=40.80.148.42 | stats count by url | sort 5 -count`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576501797/036acf68-4c6d-42e5-887e-925f3e4f37ae.png align="center")

## Challenge Question #2: Hangi IP adresi, [imnotreallybatman.com](http://imnotreallybatman.com) web sitesine karşı kaba-kuvvet saldırı gerçekleştirmiştir?

* index=botsv1
    
* sorucetype=stream:http
    
* dest=192.168.250.70
    
* http\_method=POST
    
* tarih aralığı 10 Ağustos 2016 verilen bilgilerdir.
    

Yukarıdaki bilgileri arama sorgusuna ekliyoruz. field kısmı incelendikten sonra burada önemli olan, form\_data içeriği. Kaba-kuvvet saldırısında giriş bilgileri denenmektedir.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | table form_data src_ip`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576483307/45c37556-31ba-45b7-b986-1a0e758be87f.png align="center")

stats komutu ile source ip değerler ve işlem sayıları listelenir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576467204/69d726e8-0f30-43af-99d1-cc1527c6936b.png align="center")

---

# Checkpoint #2

## Soru 1: stream:http kaynağına göre kaç tane etkinliğin http\_user\_agent içerisinde aynı anda OS X ve Chrome içermektedir ? zamanı, kaynak ip adresi ve user agent değerini içeren bir tablo oluşturun.

Verilen ipuçları,

* osx şeklinde aranırsa bulunmaz
    
* zaman (time) değeri özel bir alan
    

`index="botsv1" sourcetype="stream:http" | search http_user_agent="*Chrome*" http_user_agent = "*OS X*" | table _time src http_user_agent`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576451293/15465c51-3473-400f-8584-82311a0d5007.png align="center")

## Soru 2: stream:http kaynak türünde, joomla referansı olmayan en çok sıklıkta görülen URL nedir ve kaç kez görülmüştür?

url içerisinde joomla ifadesinin olmaması için burada wildcard kullanılıyor. sıralama url in kaç kez göründüğüne göre yapılmaktadır.

`index="botsv1" sourcetype="stream:http" NOT url=*joomla* | stats count by url | sort 5 - count`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576436248/a731f9df-503a-4841-a98c-5ba818c72891.png align="center")

## Challenge Questions #3 : kaba-kuvvet saldırısında denenmiş ilk parola değeri nedir ?

Bir önceki challenge sorusunda tespit edilen saldırıda kullanılan sorgu komutunu çalıştıralım. Listelenen değerler, varsayılan olarak en yeni aktiviteler başta listelenir. `reverse` komutu ile eskiden yeni olarak listeleme yapılır.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | table form_data src_ip | reverse`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576420470/b63cc8f8-101b-4ae5-9764-0b973894874b.png align="center")

Kullanılan başka bir metot ise `rex` komutu ile form\_data içerisinden passwd değerini çıkartmaktır. rex komutuna verilen alan içerisinde regex değerine uygun değeri ortaya çıkarabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576407340/0dd30fc2-5f4b-467b-bdc3-e060c1d1a7ca.png align="center")

arama kısmında da komut daha önce kullanılmışsa tamamlanabilir

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | table password _time src form_data | reverse`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576391910/a7671467-ec1c-4d3c-b954-0ea4219fe373.png align="center")

Ek olarak, head ve tail komutları da kullanılabilir. Arama sorgusu listelendikten sonra `tail 1` veya `reverse | head 1` komutu ile ilk kullanılan parola değeri tespit edilebilir.

## Challenge Question #4 : Kaba-kuvvet saldırısında kullanılan hangi 6 karakterli parola aynı zamanda bir Coldplay şarkısıdır?

Not: coldplay şarkı isimleri coldplay.csv içerisinde bulunmaktadır.

rex komutunu kullanarak zaten parola değerleri form\_data içerisinden ayrıştılmıştı. Bu değerlerin coldplay şarkısı olup olmadığı kontrol edilmesi gerekiyor. Karşılaştırma işleminde `lookup` komutu kullanılır. Örneğin, bir IP adresi, tehdit istihbarat ioc listesi içerisinde mi değil mi kontrol etmek için kullanılabilir.

eval komutu ile de parola değerinin 6 karakterli olup olmadığı kontrol edilebilir. Devamında search komutu ile de diğer komutlar birleştirilebilir.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | eval pass= len(password) | search pass=6 | table password src form_data`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576361283/37eaba2d-7ccc-4c00-8295-8fde74b366a3.png align="center")

6 karakterli parola değerlerini listeledikten sonra Settings &gt; Lookups bölümüne girilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576342898/40e72911-6300-4f57-b000-44ec1a309477.png align="center")

Şarkı listesinin olduğu csv dosyası yüklenir

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576329576/b9e60365-87f6-44e2-8e2f-a32a59dab8a3.png align="center")

Yapılan lab içerisinde coldpaly.csv dosyasının yüklü olduğunu **Lookups &gt; Lookup table files** kısmından görüntülenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576313846/b0488155-c8d4-48fc-9c34-f96835988d26.png align="center")

`| inputlookup cp.csv | eval song=lower(Songs) | fields song`

Arama sorgusu ile dosya içerisindeki şarkı isimleri küçük harflere çevrilir ve song alan isminde listelenir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576288476/5fba2024-6e7e-45d3-9d7e-b3321e9aea2a.png align="center")

Görselde işaretli alandan csv olarak kaydedilebilir. Burada, şarkı isimlerinin küçük harfli versiyonu coldplay.csv dosyasında bulunmaktadır.

lookup kullanım dizimi şu şekildedir, `| lookup file lookup_field AS event_field OUTPUTNEW lookup_field`

Sorgu dizinine eklenmiş hali,

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | eval len_pass= len(password) | search len_pass=6 | eval lower_pass =lower(password) | lookup coldplay.csv song AS lower_pass OUTPUTNEW song | search song=* | table password song`

Parolanın uzunuluğu belirlendikten sonra, küçük karaktere dönüştürülür ve lookup ile coldplay.csv dosyası içerisinde karşılaştırılır ve eşleşen song değeri, song isimli alana yazılır. `| search song=*` komutunun kullanılma sebebi, song field içerisindeki null değerlerini filtrelemek, not null filtrelemelerde de kullanılabilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576227319/2848c499-7870-46b3-ac31-2bae71acdf14.png align="center")

Alternatif yöntem olarak, search yerine `| sort song` eklenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576215446/fd76dfaa-e86e-4b8c-9720-d1e68ae29cfb.png align="center")

## Challenge Question #5 - Birden fazla kullanılan parola ve hangi kaynak IP adresinin kullandığını listele. Çıktı sütun isimleri src, count ve userpassword olsun.

`| stats count by field_name` olarak kullanılan komut verilen alana göre gruplayarak toplam olay sayısını hesaplamaktadır. Ancak stats sadece toplamı hesaplamak için kullanılmaz, toplama, ortalama hesaplama, standart sapma gibi hesaplamalar da gerçekleştirir.

`ndex="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | stats count values(src_ip) AS src by password | sort - count`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576197847/19c8ba43-809a-462e-891c-d49adc15c23e.png align="center")

password değerine göre IP adreslerinin gruplanmasını `| stats count values(src_ip) AS src by password` komutu gerçekleştirir ve src isminde field oluşturur.

**Ek bilgi** Eğer ki IP adreslerinin kullanmış olduğu parolaları gruplandırmak istenseydi, `| stats count values(password) AS pass by src | sort count` komut kullanılabilirdi.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576179485/aaaca4e2-6e7b-443f-bd9b-a860acf23615.png align="center")

Not: `| sort - count` büyükten küçüğe (descending) sıralama formatı, `| sort count` ise küçükten büyüğe (ascending) sıralamak için kullanılan komut.

## Challenge Question #6 : Kaba-kuvvet saldırısında kullanılan parola uzunluk ortalaması tamsayı olarak nedir?

Parolanın uzunluğu, `| eval len_password= len(password)` ile hesaplanır. Hemen ardından `stats avg(len_password) AS avg_pass` olarak eklenen komut avg\_pass isimli sütuna ortalama değerini yazar.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576147511/cf6740f5-41a7-45fb-ba5e-299ece3dcb07.png align="center")

Sonucun tam sayı bir değer olabilmesi için yuvarlama işlemi gerçekleştirilir. `round` komutu kullanılır. `eval round_avg=round(avg_pass,0)`

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | eval len_password = len(password) | stats avg(len_password) AS avg_pass | eval round_avg=round(avg_pass,0) | table round_avg`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576124452/a814cf55-0d0f-4d21-b0c8-c7fb9c2d52fd.png align="center")

## Challenge Question #7 : Kaba kuvvet saldırısında parola denemelerinin sıklığını gösteren çizgi grafiği oluşturma

çizgi grafiği `timechart` komutu kullanılabilir. Örneğin, 1 saniyede gerçekleşen parola denemelerini, zaman aralıkları sabit olan hedef IP adresleri için listeler `| timechart span=1s fixedrange=false count by dest`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576110785/d0a148a8-4f16-4b5c-b9a2-1da88efd30c6.png align="center")

## Challenge Question #8 : batman parolasının ilk kullanımı ve ikinci kullanımı arasından kaç saniye geçmiştir?

Olaylar arasındaki zaman farkını hesaplamak için `| transaction field` komutu kullanılabilir. Bu komut sonucunda duration alanı oluşturulur.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 http_method=POST form_data=user* sourcetype="stream:http" | rex field=form_data "passwd=(?<password>\w+)" | search password=batman |transaction password | table _time password src duration`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576075529/b80b03e5-cf1d-4420-8949-e6ce548e69ab.png align="center")

## Challenge Question #9: Kaynak IP adresleri ve konumlarını hesapla. Harita üzerinde ülke, şehir, enlem ve boylam bilgilerini içersin

`| iplocation field` komutu ile coğrafi konum bilgilerine erişilebilir. Maxmind GeoLite2 verileri kullanılmaktadır. `| geostats count by field latfield=lat longfield=lon` komutu, oluşan verileri harita üzerinde göstermeyi sağlar.

`index="botsv1"` [`imreallynotbatman.com`](http://imreallynotbatman.com) `dest=192.168.250.70 | stats count by src| iplocation src | table src count lat lon Country City`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724576044665/a99acaf5-441f-4060-bfb9-14b2f3745090.png align="center")

---

# Checkpoint #3

24 Ağustos 2016 tarihli olaylar için geçerlidir.

## Soru1 : Çalıştırılan en uzun komut kaç karakterlidir? İsmi nedir?

Verilen ipucu içerisinde microsoft sysmonun bir veri kaynağı olduğu belirtilmiştir. sourcetype içerisinde sysmon olan kaynak türü belirtilir. EventCode =1 filtresi sysmonda üretilen id 1 olan olaylar **process creation** (processin oluşturulması) bilgisi vermektedir.

`index="botsv1" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" | eval cmdLength=len(CommandLine) | table CommandLine cmdLength | sort cmdLength desc`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724575985665/5ac8c68d-a889-4867-a042-ba6ce3078014.png align="center")

## Soru2: we8105desk.waynecorpinc.local isimli sunucudaki top 5 windows olay kodlarını, pasta dilimi grafiğinde göster.

`index="botsv1" sourcetype="wineventlog:security" we8105desk.waynecorpinc.local | stats count by EventCode | sort 5 - count`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724575948203/7f8995f7-3ad3-4bb0-ab9c-7b4ca262b868.png align="center")

Görsel pasta grafiği için **Visualization** içerisine girdikten sonra görselleştirme türü seçilmektedir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724575918241/45531e4e-9d3f-4d52-937d-4192a64b7c69.png align="center")

---

# İlgili Kurs ve İçerikler

* Splunk, kurulumu, temel komutları ve log analizi üzerine giriş seviyesinde bir kurs: [https://learn.cyberexam.io/cyber-academy/digital-forensics/learn-splunk](https://learn.cyberexam.io/cyber-academy/digital-forensics/learn-splunk)
    
* Splunk etkili arama yaklaşımları : [https://medium.com/@efekanacar/splunkta-etkili-arama-stratejileri-02f48c0c0514](https://medium.com/@efekanacar/splunkta-etkili-arama-stratejileri-02f48c0c0514)
    
* Splunk kurulum ve log analizi için kullanımı : [https://blog.siberkulupler.com/data-analiz-canavar%C4%B1-splunk-d7db40802eb1](https://blog.siberkulupler.com/data-analiz-canavar%C4%B1-splunk-d7db40802eb1)