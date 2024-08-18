![](pics/Pasted%20image%2020230117142142.png)


![](pics/Pasted%20image%2020230117142518.png)

Uygulama ekranında kullanıcı input alanı ve buton bulunmaktadır. Uygulama işleyişi gözlemlenebilir. Butona basıldığında NOPE olarak uyarı mesajı yayınlıyor. 

jadx ile uygulama kodları incelendiğinde, MainActivity içerisinde onClick metodu içerisinde flag değeri ile bağlantılı getFlag() fonksiyonu çağrılmaktadır.

Fonksiyon içerisinde, kullanıcıdan gelen input değerinin belirlenmiş parola değerine eşit olup olmadığı kontrol edilmektedir. Eşitliğin sağlanmaması durumunda "NOPE" mesajı yayınlanmaktadır.

![](pics/Pasted%20image%2020230117143152.png)

Parola değeri resource dizini altında  strings.xml dosyasında bulunur.

![](pics/Pasted%20image%2020230117143608.png)

![](pics/Pasted%20image%2020230117143711.png)

Bulunan değeri uygulama içerisinde input alanına girelim. Flag değeri ekranda gösterilmektedir.

![](pics/Pasted%20image%2020230117143905.png)