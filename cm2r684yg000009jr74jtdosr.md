---
title: "MITRE: Discovery: T1040"
datePublished: Sun Oct 27 2024 05:50:18 GMT+0000 (Coordinated Universal Time)
cuid: cm2r684yg000009jr74jtdosr
slug: mitre-discovery-t1040
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730008680780/102cee8a-7ed0-41b2-91b2-8ed4b3ddeaf6.png
tags: network, discovery, cyberexam, mitre-attack, sniffing

---

Lab linki: [https://learn.cyberexam.io/challenges/mitre/discovery/mitre-discovery-t1040](https://learn.cyberexam.io/challenges/mitre/discovery/mitre-discovery-t1040)

---

## Görev Tanımı

Tehdit aktörleri hedeflediği sistem hakkında bilgi almak için ağ trafiğini pasif olarak izlemektedir (sniffing). Kimlik doğrulamanın nasıl yapıldığı, çalışan servisler ve versiyonları, ağ içerisindeki cihazlar, kullanılan iletişim kanalı gibi bilgilere erişilebilir. Ağı dinlemek için wireshark, tshark, tcpdump araçlar kullanılabilir.

Bulut ortamında da traffic mirroring adı verilen servisler (AWS Traffic Mirroring, Azure vTap) kötüye kullanılabilir. Bu araçlar ağ trafiğinin analizi ve performans sorunlarının çözümü için trafiği başka bir cihaza yönlendirmektedir. Load balancer seviyesinde TLS termination uygulanırsa gönderilen trafik açık metin şeklinde şifrelenmeden gönderilecektir.

Saldırı makinesi erişim için `ssh -i victim_key victim@attack`

Tespit makinesi erişim için `ssh -i admin_key admin@detect`

## Lab Çözümü

### Saldırı makinesine bağlanın ve iç ağ trafiğini dinleyin. Kurban makineye bağlanan istemcinin IP adresi nedir?

İlk olarak dinleyeceğimiz ethernet arayüz ismini `ip a` komutunu kullanarak öğrenelim. Ethernet arayüzü belirtmezsek tümünü dinleyecektir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730004937592/1bef8d4d-d194-4366-9692-764333bba765.png align="center")

Kullanılan diğer bir yöntemde ise `tcpdump -D` komutu ile de ağ üzerinde dinlenebilecek arayüzler listelenebilir.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730005452902/a937d00e-27fd-4a70-bef3-5cb293036299.png align="center")

tcpdump aracını kullanarak ağ paketlerini yakalayalım. `tcp -i eth1 -w captured.pcap` komutu ile yakalanan ağ paketlerini captured.pcap dosyasına kaydedelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730005196292/b6ab6521-81b5-4b5a-93fb-b79597f3171c.png align="center")

PCAP (Packet capture), belirlenen arayüzün ağ trafiğini içeren dosya türüdür. Okumak için tcpdump aracını kullanabiliriz. `tcpdump -n -r captured.pcap` komutu ile içeriğini görüntüleyelim.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730006316384/b7ebf0eb-351c-4e6f-8f83-b2a8f0d175ee.png align="center")

Formatı incelediğimizde zaman, kaynap IP.Port &gt; hedef IP.Port, bayrak değeri, sıra numarası ve options bölümü bulunur. Kaynak IP adresi ile hedef IP (yani kurban makinenin adresi 192.168.1.40) arasında üçlü el sıkışma gerçekleşmiş ve TCP bağlantısı kurulmuştur.

### Bağlanmak için kullandığı protokol nedir?

Saldırganın bağlantı kurduğu port numarası 21 dir.

### Kullanıcı adı? Parolası?

Kullanılan protokolden kaynaklı olarak ağ trafiğinde kullanıcı adı ve parola değeri açık metin olarak gösterilmektedir.

### Tespit makinesine bağlanın ve ağ dinleme işlemi ne zaman başladı?

/var/log/messages dosyası içerisinde sistemle ilgili mesaj kayıtları bulunur. Burada ağ dinleme işleminin belirtildiği ifade arayüzün promiscuous moda geçmesi ile gerçekleşir. Bu mod serbest dinleme modudur ve ağ trafiğindeki tüm paketlerin yakalanmasını sağlar.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730007164137/a2740cf5-edb1-4569-abb7-4f698d6853f5.png align="center")

/var/log/audit/audit.log içerisinde de filtreleme yaparak yine dinleme işleminin ne zaman gerçekleştiğini bulabiliriz. `grep "tcpdump" /var/log/audit/audit.log` komutunu çalıştırarak saniye formatında zamanı bulabiliriz. Uygun formata çevirmek için `date -d @<seconds>` komutu kullanılır.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730007410813/6499fc21-1452-4457-8fd2-27c7ee54179b.png align="center")

### Sniffing işlemi kaç saniye sürdü?

/var/log/messages log içerisinde ağ trafiğini dinleme işleminin başlama ve bitiş süreleri veriliyordu. Bunlar arasındaki zaman farkı kolay bir şekilde hesaplanbiliyor. Ancak chatgptye bunu hesaplatan kodu yazdırabiliriz. Verdiği komut,

```bash
awk '/promiscuous mode/ {print $1, $2, $3}' /var/log/messages | awk 'NR==1{start=$0} END{end=$0; cmd="date -d ""start"" +%s"; cmd | getline s; close(cmd); cmd="date -d ""end"" +%s"; cmd | getline e; close(cmd); print "Sniffing Süresi: " (e-s) " saniye"}'
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730007642433/53af63bc-a2a3-4900-a92f-99e8a35d515d.png align="center")