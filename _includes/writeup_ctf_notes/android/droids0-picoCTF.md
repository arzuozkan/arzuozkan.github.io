![](pics/Pasted%20image%2020230116131808.png)

uygulama dosyası zero.apk. Burada flag değerinin uygulama loglarında olabileceğini anlayabiliriz yapılan tanımdan. 

apk dosyasını emülatore yükleyip açabilirsiniz veya jadx ile kod analizi gerçekleştirebiliriz.


![](pics/Pasted%20image%2020230116142322.png)

MainActivity içerisinde butona basıldığında getFlag() fonksiyonu çağrılıyor.

![](pics/Pasted%20image%2020230116150712.png)

fonksiyona çift tıklatıldığında tanımlandığı sınıfa

![](pics/Pasted%20image%2020230116151259.png)

flag değerini native kütüphaneden yükleyip değeri log olarak yazdırmaktadır. 

> adb logcat 

komutu ile uygulama logları görüntülenir. log için verilen tagi grep ile filtreleyip listeleyebiliriz.

![](pics/Pasted%20image%2020230116152308.png)


# Extra

Main activity onCreate içerisinde ==hellojni==  isimli native kütüphane yükleniyor. JNI (Java Native Interface) sayesinde c/cpp kodları yüklenebiliyor. Native kodlar lib/ dizini altında .so uzantılı dosya olarak bulunur.

![](pics/Pasted%20image%2020230116152939.png)

apktool aracı ile apk dosyası smali kodlarına çevrilerek açılabilir. d, verilen apk dosyasını decode eder. b ise açılan paketi tekrar build etmek için kullanılır.

> apktool d zero.apk

lib/ dizini altında birçok mimari için .so dosyası bulunur.

![](pics/Pasted%20image%2020230117134740.png)

disassembler ile .so uzantılı dosya açılabilir. Bu adımda Ghidra aracını kullandım. Çağrılan fonksiyon paprika() isimli. 

![](pics/Pasted%20image%2020230117134549.png)

not: buradan reverse yapılır mı devamını getiremedim :(