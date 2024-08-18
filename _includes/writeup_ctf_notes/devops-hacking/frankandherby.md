# Frank & Herby make an app - THM WriteUp
tryhackme room: https://tryhackme.com/r/room/frankandherby

## Nmap Taraması
nmap taramasında 22, 3000 ve 31337 portlarının açık olduğu görülmektedir. Kubernetes ortamlarında uygulamalara dışardan erişebilmek için NodePort servisini kullanarak 30000 - 32767 portları arasında yayınlanır. 

![image](https://github.com/user-attachments/assets/959138f5-8462-47a1-aa5d-6aaaa149e113)

31337 portunda nginx 1.21.3 sürümünü kullanan bir uygulama çalışmaktadır.

![image](https://github.com/user-attachments/assets/c6adc41e-560a-4eb0-8f64-3375b9717c46)

3000 portunda rocket.chat isimli bir uygulama çalışıyor

![image](https://github.com/user-attachments/assets/91ce1122-7e82-430d-83af-2a8359b7c555)

rastgele bir kullanıcı hesabı oluşturup içerisine girildiğinde geliştiricilere ait bir yazışma görülmektedir. 

![image](https://github.com/user-attachments/assets/72ec12a1-8b0f-4d5e-b3db-8c2df5c090e0)

![image](https://github.com/user-attachments/assets/4770b0c5-05f9-4865-b07e-47d7bf75c8a7)

uygulama için git versiyon kontrol sistem aracını kullandıkları anlaşılabilir. Bir repository oluşturmuş ve bunu açıkta bırakmış olabilir. Dizin taramasını yaygın olarak kullanılan wordlistler ile istenen dizin tespit edilemedi. Daha çok gizli dizinleri barındıran bir wordlist kullanılması gerekiyor.

![image](https://github.com/user-attachments/assets/bacabcf9-8280-4b58-afd7-65e62c476204)

Dizin içerisinde giriş bilgileri yer almaktadır. Verilen giriş bilgilerini URL decode ile çözülmesi gerekebilir. Bu bilgiler ile SSH oturum giriş bilgileri olarak da kullanılıyor. frank sudoers içerisinde yer almıyor ve yüksek yetkilerde değil. İlk user flag dosyası frank home dizini altında yer almaktadır.

![image](https://github.com/user-attachments/assets/97e303b2-192c-41b4-8786-48204eb08a05)

## Yetki Yükseltme
### Pod Escape to Host

geliştiricilerin konuşmaları içerisinde microk8s aracı ile kubernetes ortamının oluşturulduğu ve `microk8s kubectl` komutu ile çalıştırıldığı yer almaktaydı. `microk8s kubectl auth can-i --list` komutu ile, içerisinde bulunulan bağlamda hangi yetkilere sahip olunduğu öğrenilebilir.

![image](https://github.com/user-attachments/assets/cf6f9717-585a-48a5-bdcc-9a28ae24ad58)

Çıktının yorumlanması gerekirse kısaca, tüm kaynaklar için, tüm yetkiler verilmiş. 
localhost:32000/bsnginx image kullanarak hosta erişim için bir pod oluşturalım. Bu image çalışan pod içerisinde kullanılan image ve bunu öğrenmek için `microk8s kubectl get pod POD_NAME -o yaml` ile görüntülenebilir. Kullanılan tek satırlık kod ile yetki yükseltilebilmektedir. 

![image](https://github.com/user-attachments/assets/6f48cb41-ea57-4b76-8cd8-8163fd69289e)

### Host Mount to Pod
İkinci bir çözüm yolunda ise host makinesinin root dizini bir pod içerisine mount edilip, pod içerisinden erişim gerçekleşebilir. Bunun için bir yaml dosyası oluşturulur.

![image](https://github.com/user-attachments/assets/7301a845-0efe-4c7e-8a7c-1e996834cfdd)

Sonra, `microk8s kubectl apply -f root-pod.yaml` komutu ile pod oluşturma işlemi gerçekleştirilir. `kubectl get pod` komutu ile de hatasız bir şekilde çalışmaya başladığı kontrol edilebilmektedir.

![image](https://github.com/user-attachments/assets/3bd8db5e-c36b-459b-837a-7b3a47d667b3)

`microk8s kubectl exec -it hostmount -- /bin/bash` komutu ile pod içerisine erişilir. Burada host makinesinin root dizini /opt/root içerisine mount edilecek şekilde ayarlanmıştır. Bu yüzden root.txt dosyası root dizini altında bulunduğundan /opt/root dizini host makinesinin / dizinini göstermektedir.

![image](https://github.com/user-attachments/assets/7ef293cb-0929-4c3e-a30c-77e373c8ce5d)

### note
Farklı çözüm yöntemlerinde gördüklerim, kubernetes enumeration araçları kullanarak, zaten sistemde varolan  yetkilendirilmiş (privileged) pod tespit ederek bu pod üzerinden işlem yapılabilir.

![image](https://github.com/user-attachments/assets/f7b265dc-2cf7-40ac-a07c-96a99cb31a1c)

-A parametresi oluşturulmuş tüm namespaclerdeki podları listelemek için kullanılır. Burada diğer namespaceler içerisinde çalışan podları araştırıp yetkileri ve ayarları kontrol edilerek çözüme gidilebilir. Ancak, yetki kısıtlaması olmamasından dolayı bu adıma gerek duyulmamaktadır.

## Kaynaklar
- https://exploit-notes.hdks.org/exploit/container/kubernetes/microk8s-pentesting/
- https://owasp.org/www-project-kubernetes-top-ten/2022/en/src/K04-policy-enforcement
- https://www.container-security.site/general_information/tools_list.html


