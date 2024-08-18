# Kubernetes For Everyone - THM Writeup
kaynak: https://tryhackme.com/r/room/kubernetesforyouly

## Access The Cluster
### Açık Portların ve Servislerin Keşfi
Kubernetes cluster erişimi için nmap ile çalışan servislerin ve açık portların keşfi yapılmaktadır. İlk aşamada, basit bir servis taraması yapan bir komut çalıştırılabilir. `nmap -sS -sV IP_ADDR -Pn` taramasının sonucunda 22, 111, 3000 ve 5000 portları açık olarak bulundu.

![image](https://github.com/user-attachments/assets/f285fdf9-2f24-4d56-9726-c32ce1cd62d1)

Kubernetes API ve servislerinin kullandığı bazı varsayılan port numaraları bulunmaktadır. Örnek olaraka, 443,6443,8443 gibi portlar Kubernetes API portu olarak kullanılmaktadır, 30000-32767/TCP port aralığı Kubernetes servisinin çalıştığı port aralığıdır.
`nmap -n -T4 -p 443,2379,6666,4194,6443,8443,8080,10250,10255,10256,9099,6782-6784,30000-32767,44134 IP_ADDR -Pn` komutun çıktısında 6443 portunun açık olduğu tespit edilmiştir.

![image](https://github.com/user-attachments/assets/ae126b58-3565-48e0-b8ea-9dee74b0896c)

5000 portunda bir Python Flask uygulamasını inceleyelim. 

![image](https://github.com/user-attachments/assets/fd6d16f4-4369-4aba-8804-63fb92c4411c)

Kaynak kodlarına bakıldığında, submit butonunun bir işlevi yok sadece boyanan kareleri sıfırlıyor. script.js ve main.css dosyaları bulunuyor. main.css içerisinde bir pastebin bağlantısı verilmiş.

![image](https://github.com/user-attachments/assets/933420ca-4b4e-4e4f-99a7-84fc36b3e368)

Bağlantıya gidildiğinde base64 encode edilmiş bir metin bulunuyor

![image](https://github.com/user-attachments/assets/193d99ff-33db-4029-8f8f-387b3a148ada)

cybercef aracını kullanarak decode edildiğinde base32 ile encode edilmiş olduğu ve decode edilmiş metin bulunmaktadır. Bu sayede kullanıcı adı bulunmuş oldu.

![image](https://github.com/user-attachments/assets/a2d5424a-f73d-4064-95db-b85d8e395c3e)

Werkzeug / Flask uygulamalarında eğer debug mode açık bırakılmış ise /console dizine erişim mümkün olabilmektedir. PIN korumalı olsa da exploit edilme ihtimali bulunur. Werkzeug PIN exploit gibi birkaç exploit denedim ancak exploit edemedim. 
Diğer portlara bakalım 3000 portu açıktı. Grafana login sayfası ile karşılaşıyoruz. Burada versiyon bilgisine erişebiliriz. Grafana 8.3.0

![image](https://github.com/user-attachments/assets/c311ff1c-8797-40c3-95b3-153557c31b62)

### CVE-2021-43798 Grafana Directory Traversal and Arbitrary File Read

/public/plugins dizini üzerinden path traversal yaparak dosya okunabilme zafiyetidir. Basit bir payload ile /etc/passwd dosya içeriği okunabildiği görülmektedir. 

![image](https://github.com/user-attachments/assets/7d1d9b47-1ad4-4b02-b1b0-ce33431db8c4)
![image](https://github.com/user-attachments/assets/3ce82ab4-699a-42c4-821a-83ae9adcd341)

Burada da istenilen parola değeri bulunmuş oldu. CVE-2021-43798 zafiyeti için birçok exploit kodu mevcut. Burada kullanılan, [taythebot/CVE-2021-43798](https://github.com/taythebot/CVE-2021-43798) golang ile geliştirilmiş bir araç. Kullanımı incelendiğinde dosya okumanın dışında veritabanı ve yapılandırma dosyalarını dump edebilme gibi özelliği mevcut
`go run exploit.go -target http://10.10.125.83:3000 -dump-config -output defaults.ini` komutu ile grafana yapılandırma ayarlarına erişilebilir. Bu dosyayı incelerken varsayılan giriş bilgilerinin değiştirilmediğini görebilirsiniz. admin:admin bilgileri ile grafana arayüzünden giriş yapılabiliyor.

Açık portlar içerisinde 22 SSH portunun da açık olduğunu hatırlayalım. ssh anahtarına erişim sağlarsak shell alabilirdik belki ancak erişilemiyor. 

![image](https://github.com/user-attachments/assets/5a126c93-d741-4856-8b4f-d4cf5afe1d32)

Bu zafiyeti kullanarak nasıl sisteme erişim sağlanır, onu bulmamız gerekiyor. Ancak zafiyet uzaktan kod çalıştırma (RCE) olmadığı için bu amaçla kullanamadım. Bu yüzden bir önceki bulduğumuz kullanıcı adı ve parola değerlerini ssh bağlantısı için kullanalım

![image](https://github.com/user-attachments/assets/d8a50a85-0ae6-4a11-8c27-7b3df25da676)

vagrant kullanıcısı olarak bir oturum elde edildi. `sudo su` komutunu çalıştırınca root yetkilerine sahip olundu. /etc/passwd dosyası içeriğine bakıldığı zaman kubernetes servislerinin çalıştığı anlaşılabilir. Benzer şekilde sistemde çalışan uygulamalar kontrol edilebilir.

![image](https://github.com/user-attachments/assets/f4704d5e-2e7e-4d97-872f-3aa2dbc0c225)

## Find Secret
İstenen bir secret değeri var ve kubernetes ortamında hassas veriler için secret objesi kullanılmaktadır. k0s, kubernetes ortamını kurmak için kullanılan araçlardan birisidir. Kullanımı için [k0s dokümantasyonu](https://docs.k0sproject.io/stable/) incelenebilir. `k0s kubectl get secret` komutu ile secret objeleri listelenmektedir. (İlk başta iletişim sağlanmadı gibi hata mesajı alabilirsiniz, lab ortamından dolayı çalışan servislerle alakalı olabilir ve bazen 6443 portu kapalı olabilir)

![image](https://github.com/user-attachments/assets/311be612-ac45-4a0d-99a5-f489b89d2013)



## Kaynaklar
- https://cloud.hacktricks.xyz/pentesting-cloud/kubernetes-security/pentesting-kubernetes-services
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug
- https://vulncheck.com/blog/grafana-cve-2021-43798
- https://github.com/taythebot/CVE-2021-43798

