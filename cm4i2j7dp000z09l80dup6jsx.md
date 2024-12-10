---
title: "MITRE: Defense Evasion: T1027.001"
datePublished: Tue Dec 10 2024 06:16:25 GMT+0000 (Coordinated Universal Time)
cuid: cm4i2j7dp000z09l80dup6jsx
slug: mitre-defense-evasion-t1027001
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732732107373/9de78f85-a868-469e-850e-69e2930b5f38.png
tags: cyberexam, mitre-attack, defense-evasion, binary-padding

---

Lab: [https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1027-001](https://learn.cyberexam.io/challenges/mitre/defense-evasion/mitre-defense-evasion-t1027-001)

---

## Görev Tanımı

Bu teknikte saldırganın amacı zararlı yazılımın içeriğini maskelemek ve karmaşıklaştırmasıdır. Binary padding olarak bilinen bu teknikte, zararlı yazılıma önemsiz verileri (junk data) ekleyerek boyutu değiştirilir. Böylece zararlının içeriği değişmese de hash değeri değişebilir. Hash tabanlı tespitlerde kaçınılsa da diğer tespit yöntemlerinde işe yaramayabilir. Aynı zamanda tersine mühendislik işlemlerini de karmaşıklaştırmayı hedefler. Başka bir amacı da, dosya boyutunu büyüterek sandbox gibi analiz ortamlarının analiz yapmasını engellemektir.

## Lab Çözümü

### Null-padded-binary dosyasını analiz edin ve dosyanın orjiinal md5 hash değeri nedir?

Dosya türünü kontrol edelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732733456099/3aa654be-9d22-4c17-8a60-94c346f4e38f.png align="center")

`hexdump` komutu dosyanın hexadecimal gösterimini sağlar. -C parametresi ile ascii karşılığını da görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732718672785/cfbac615-79c1-4f75-8604-56a1e2e8e758.png align="center")

radare2 disassembler aracı ile de zero byteları görebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733298577029/96fc44dd-c787-41db-b870-bf0a0bd36e44.png align="center")

Burada dikkatimizi çeken şey dosyanın son kısmında çok fazla 0(zero veya null) byteların olması. Bu teknik, Null-padding veya zero-padding binary olarak isimlendiriliyor. Basit bir python scripti ile null byteları silebiliriz.

```python
def remove_null_padding(input_file, output_file) :
    with open(input_file, 'rb') as f:
        data = f.read()
    trimmed_data = data.rstrip(b'\x00')

    with open(output_file, 'wb') as f:
        f.write(trimmed_data)

remove_null_padding('binary_file', 'cleaned_file')
```

Farklı yaklaşımlarda mevcut tabii, örneğin `truncate` komutu ile dosyayı orjinal boyutuna getirebiliriz veya `dd` (disk dump) komutu ile kopyalama işlemi yapılabilir. Burada bulunması gereken 00 olmayan son byteın indexi. Chatgpt kullanarak oluşturulan bir python scripti ile buldum

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733295668317/a40284de-9400-4e32-90e7-d8d046976b30.png align="center")

Sonrasında dd komutu ile orjinal dosyayı oluşturmaya çalıştım. `dd if=null-padded-binary of=original-binary bs=1 count=last_index+1` . Komut içerisinde bs=1 olarak ifade edilen blok boyutunu 1 byte olarak tanımlıyoruz. Varsayılan değeri 512byte.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733296094070/5d821080-d17c-4fef-88d2-af15ee719aba.png align="center")

`md5sum original-binary` komutu ile MD5sum hash değerini görüntüleyebiliriz.

### Random-padded-binary analiz edin ve orjinal dosyanın hash değeri nedir?

Random-padding olduğu zaman işler biraz daha karmaşıklaşmaya başlıyor. Burada dosyanın sonu, başı veya belirli bir kısmına eklemeler (padding) yapılabiliyor. İlk incelediğim zaman hiçbir şey anlamadım. Düzenli bir ifade mi bulmam gerekiyor veya örüntü şeklinde mi ekleme yapılmış bunu araştırmamız lazım.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733296677138/ac92af06-9d23-447f-bb4c-1da2c874752b.png align="center")

Benzer şekilde hexdump ile açtığım zaman açık olarak göremedim padding kısımları. r2 aracını kullandım ancak onunla patchleme işlemini halledemedğim için arayüze sahip cutter aracını kullanacağım. Dosyayı write modda, `cutter -w random-padded` komutu ile açıyorum. Ben bu işlemi yapmadan önce `random-padded-binary` dosyasının kopyasını alıp onun üzerinde çalışmalar yapacağım.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733477025526/c1e38be3-37ba-4530-b618-8c4627025247.png align="center")

Bu aracın bir özelliği seçilen kısım hakkında hash bilgilerini sağ taraftaki arayüzde gösteriyor. Paddingleri seçmeden dosyanın orjinal olduğu kısmını seçtiğimi varsayıyorum.

### Varyasyonları?

Dosyanın orjinal halindeki hash bilgisini bulduktan sonra varyasyonlarını manuel olarak tespit etmeye çalışalım, sonundaki bir zero byte dahil ediyorum. Tespit edilen zararlının hash bilgisine ulaşmak için birkaç tane de ekleyebiliriz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733477569402/22f0a74d-18f6-4e5c-a611-7b773e442c1c.png align="center")

### Zararlı yazılımın ismi?

Bulduğum bu hash bilgisini Virus Totalde arayacağım. Daha öncesinde analizi yapılmış ve 36/65 tespit oranına sahip bir zararlıya ait olduğunu tespit ediyoruz.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733477687510/51a59c81-e47e-45f8-9cf0-d36a828ae328.png align="center")