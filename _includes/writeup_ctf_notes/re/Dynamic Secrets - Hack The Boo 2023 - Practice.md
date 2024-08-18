Verilen dynamic_secrets dosyasının türü ELF executable. Linux işletim sistemlerinde çalıştırılabilir olduğunu göstermektedir.

![](../pics/Pasted%20image%2020231026091558.png)

Programın ne yaptığını anlamak için çalıştırıyorum. Parametre olarak bir parola değeri aldığını görebiliriz.

![](../pics/Pasted%20image%2020231026092020.png)

Reverse için radare(r2) veya ghidra kullanabiliriz. İlk olarak r2 kullanımı zor olduğundan onu kullanacağım.

> `r2 -d dosya_ismi`

komutu ile dosyayı debugger mode olarak açalım. Bu parametre dosyayı çalıştırarak analiz yapabilmemizi sağlar.

> `aaa` komutu ile otomatik analiz işlemleri başlatılır ve `afl` komutu ile de dosyanın içerdiği tespit edilen fonksiyonlar listelenmektedir.

![](../pics/Pasted%20image%2020231026093018.png)

Bilinen fonksiyonların dışında, geliştiricinin kendi yazmış olduğu fonksiyonlar challange çözmek için yardımcı olacaklar.

> `pdf @main`  komutu ile verilen adres veya fonksiyon ismindeki assembly kodlarını göstermektedir.

Main içerisini inceleyebiliriz. Genelde programın başlangıç noktasıdır. Eğer main yoksa farklı yöntemler kullanarak programın entry pointini bulmamız gerekir.

