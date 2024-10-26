---
title: "AWS ile Wordpress ve MySQL Kullanımı"
datePublished: Sat Oct 12 2024 21:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2nip58n000g09mkdl7ue6ca
slug: aws-ile-wordpress-ve-mysql-kullanimi
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729982136463/33286341-dc4b-411d-9905-5e4acbb2dcbf.png
tags: aws, wordpress, devops

---

## 0\. Proje Mimarisi

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfsFT4VQ4hruMJjcTE4zUyL2EtazlHfz59sx1ikwVBNlGwPvAg7Ad5e70sX5Ftt1fDm4asBjkJH-TIj-HFn8PgT6yZzHlTn7aU5d5Ii-J7J-w1UL4p1HPPSHQIVIKnRxS2jMpS-R6ONWEI-SOAriWMg2Mw?key=lzOqzu63NCHzpfnZe8V--g align="left")

## 1.Aşama: AWS üzerinden EC2 servisi ile Ubuntu sunucusu kurma

AWS Console üzerinden giriş yaptıktan sonra arama bölümüne EC2 yazarak servise erişebiliriz. instance imajı, türü, anahtar çifti gibi kurulum için gerekli bilgileri giriyoruz.  

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf0GPt-hL5WNM-UvTR461bBykWOaFA0iST2z-Srq9MOooyKJS9bEArgw44qdeQSZ1n1z_u5AqfJyu07zM2ccsGIGXN3fZ_2G6kZeYBNNejYX_o59yNawuo2ollzKeFY8h9UratgcTZOylQ0t5zoG7XSGWlj?key=lzOqzu63NCHzpfnZe8V--g align="left")

Ardından **launch instance** seçeneği ile sunucu oluşturmaya başlayalım. Sunucumuz oluşturulmuş durumda instances kısmından listelenmektedir.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdcnmvCHaXjUKpTfE6EMq6yei3Z7STGMxWGBZIR871RJcSMcAeBNUh0hTMx8xtfMTQ51cf9qONY4vpn85CTHB7g-uejr0it2c7htfbijdZeL3p2rH3wQsHkPbjhNQpkTeeg8rvV8BzMIBJ1Pi5AA9s7_7hp?key=lzOqzu63NCHzpfnZe8V--g align="left")

Yukarı kısımda bulunan Connect seçeneğine tıklayabiliriz veya kendi bilgisayarımızdan erişmek istersek de oluşturmuş olduğumuz anahtar çiftini kullanarak ssh bağlantısı yapabiliriz.

## 2.Aşama: RDS servisi ile MySQL veritabanı kurulumu ve entegrasyonu

Arama kısmına RDS yazarak veritabanı servisine erişebiliriz. **Create Database** seçeneğine tıklayalım

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXd4348nUj9OIlW-vQ0Y2g-qEvxeHQz-w0kEch8hRl3ZwP6F5qsd8H1ncvcs7CDOLE-VJv6Lu-AldSCfe1ztgEX2tO0-2_hEFm3UEP7Kvh5cY7jkuXGkgGF3TN_kYHuUISNM0-ycoM2LmgOSBj61dkKgFws?key=lzOqzu63NCHzpfnZe8V--g align="left")

Standart kurulumu seçiyoruz. Ardından kullanacağımız veritabanı seçelim burada MySQL işaretliyoruz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeRsG6jj9DKN21tw0ue5DwVf-cWzo2bqINEiYLRmg-MveUrUxp4nZFfNj1rV1jRuZ6jawmke564bq-OB6pSF-6PsiB7XxgXzHHxqyJsU5CM1K4zcduC7Hnkpl4r4tqQPY4BJoMphoUM_hEgn66izkSjcCTD?key=lzOqzu63NCHzpfnZe8V--g align="left")

Veritabanı ismini ve kullanıcı ismini belirtiyoruz

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXekKqIfyGZtQ0TlDvDQxBG4gLZTWmIorRoABhWykiRuJbs2iXQdWJV0zC1xk_IC_exptOK2oo4Rcb8ptnBM0Kp9uEluhz3T8RpcKjS8n88lmGKY-cw_1lCsm0mEih16-nX_XIeZwe1DlSADxv8tlT5FfjOW?key=lzOqzu63NCHzpfnZe8V--g align="left")

Sonrasında Connectivity kısmından da wordpress kuracağımız EC2 ubuntu sunucumuzu bağlayalım.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdzQo8eZce0-sEsWoY-WGo27LeAfF9aobnF36jA8nos6S_4hv5D4yvAM06WBHqzuedK1F5IpedEwYLzhda5cTdzyQSxVtOb17MtbsVhwdVSTZROzBnISQ3V48DkmQmVd1xMCLKY2PlASZmLWZZjzQO2egRw?key=lzOqzu63NCHzpfnZe8V--g align="left")

En alt kısımda **Additional Configuration** kısmından yeni bir veritabanı oluşturalım

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXec0Wr4y3lRxaGeLDJMfjb-ZRUn2xI_rrmbI_Dthp287TqF9ldmT7F269eI3Mt-rFuCWhAo1eJSQpYQDeCJRLO2JE4QVrTZ5A2eEYXQUVfb9G08f3eIxP4kEndlkSn-2Zy1XW2hP0jiu_sXxaVHmb-Z3lJ5?key=lzOqzu63NCHzpfnZe8V--g align="left")

Tüm bilgileri girdikten sonra veritabanımız oluşturulmaya başlanıyor. Status kısmında **Creating** yazıyor. Veritabanı kurulumu tamamlandıktan sonra o kısım **Available** olarak görünecektir. 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcrMrCNQPFt87Dbb5I7lKuAJIBCoupK1MMGxUWg2s8Em75fb1j9U4ciFSxrx-D-XuLlHPo51tSSsz1rtvCRBjMQQ-tKDVDHHPPmOrh7XsQYTVUEfSSIqZ5u91QXx8WKBzsXeGY44Y_6qSWZvTzI_SW5vfEC?key=lzOqzu63NCHzpfnZe8V--g align="left")

## 3.Aşama: Wordpress Kurulumu

Sunucuya erişim sağladıktan sonra Wordpress kurulumuna geçelim.Sırasıyla aşağıdaki komutları çalıştıralım.

```bash
sudo apt update  
sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl status nginx
sudo apt install php-fpm php-mysql php-xml php-curl php-mbstring -y
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo mv wordpress/* /var/www/html/  
sudo chown -R www-data:www-data /var/www/html/  
sudo chmod -R 755 /var/www/html/
```

Ubuntu EC2 sunucumuzda çalıştırıyoruz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeZVYMlchADcxR4heXm3I457qUwZ5taqwxBtBP5QVu3DeBgeBPjU3WNHhGxJ4QIOXU6QX4zVJ24pG3GS2vK5NlMOQLKilxPp6mhQiW6FjHzrKP6o_Na85KmAasLSuMGvhT3c4i6SN7wt1V85cVrr9TODHQr?key=lzOqzu63NCHzpfnZe8V--g align="left")

nginx kurulumunun doğru bir şekilde gerçekleştiğini `systemctl status nginx` komutu ile öğrenebiliriz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeKrg4NO2VcU5muQl5iYR5culnMrg4tbox7gC8Zkwp-vxcQGdQxenue21wGVciuxPW7o4nfUIUumfhkhE0g3xMMOSMFX3cCD6j-LIBdi9WGj0oUJ7BRdZ4ZmQVzpEoYtVGWj3ueVYfmaU2kudi2Bzt8tEI?key=lzOqzu63NCHzpfnZe8V--g align="left")

wordpress isimli dosya oluşturalım.

`sudo nano /etc/nginx/sites-available/wordpress` 

komutunu çalıştırıp dosya içeriğini ,

```xml
server {  

    listen 80;  

    server_name EC2_IP ; # Burayı kendi alan adınız veya IP adresinizle değiştirin ;

    root /var/www/html;  

    index index.php index.html index.htm;  

    location / {  

        try_files $uri $uri/ /index.php?$args;  

    }  

    location ~ \.php$ {  

        include snippets/fastcgi-php.conf;  

        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # PHP versiyonunu kontrol edin  

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  

        include fastcgi_params;  

    }  

    location ~ /\.ht {  

        deny all;  

    }  

}
```

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcazAxfU0jGC6e5ebKpwVoaBwwMAzvVY1TSvnn8qTJ4sfKnWhzIAikaykA6_NNEzmDiQsbMxJrydCH3jmwTp6xdcXao1pf904as6cghNH_ScPBEavTXVKrrRtC0u9aQ7zQjT-gtCzgdl4Q5FGS6kydBdzwJ?key=lzOqzu63NCHzpfnZe8V--g align="left")

Yapılan ayarların kaydedilmesi için aşağıdaki komutları çalıştıralım. Bu komutlar, konfigürasyonları uyguladıktan sonra servisleri tekrardan başlatıyor.

`sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/`

`sudo systemctl restart php8.3-fpm`  

`sudo systemctl restart nginx` 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfIxjnqvR3hrM_0R9hetIdRfOTm69eE-hekBgmspEpkJKybKN-ak4XAJppAXHDH8H52xG2gCmpvvmQs2c392kQ0ArHlPLG6-His2s8_gK69k9foOrrx21wuRFCRV9BoiTz7oWYpNFlk5lOZmlDwu7M14gM?key=lzOqzu63NCHzpfnZe8V--g align="left")

Bu işlemlerden sonra veritabanı yapılandırması için RDS Endpoint ve port bilgisini alıyoruz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfQ4pMX7p29g7pHFkC9fOLTmBmGksNjgEWmY8UIa9zvo_U25stu0CRh5sgCwK-tGJ583w2yQv__EPrxkXqz0uEKgGevQYmkWlYVJZ7M4c3MgwFQ4qB4WkgaYNBulvM5RQPQ6FGxdMT2uf5k8zWIUqOgWcod?key=lzOqzu63NCHzpfnZe8V--g align="left")

/var/www/html dizini altında wordpress dosyaları bulunuyor. wordpress-config-sample.php dosyasına veritabanı ile ilgili bilgileri giriyoruz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcqD0KtjkvroibogQ1wB2nUTlBAEGwGFYQV2LyKZ3kc1hArF-BUC35Za-WqUKcLQIfUyBMmwEZKqD0WNUtpsfQzxUiv5zVTxVEB3Mt7uP_HJDQzzsz_GMdW1UTfKAltCoDPCPe-JBxVZAKSEmiAQnscNE0n?key=lzOqzu63NCHzpfnZe8V--g align="left")

EC2 sunucusu Inbound rule 80 portunun erişime açık olması lazım kontrol edebiliriz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXf2J7Datfm5PPY7pKCKOf7SyywDuo25dyvf2bN3WADjqHaUo06Bc-jtSYB_tKoJrFRj_lCXPTMy1Tg4bnNGMBHZWGezRwrtUiRwx79s6e6oA2yxPsDJ9R6mrw4UMo_EGkfjHhe2xR_rS42553xC89YCJUS7?key=lzOqzu63NCHzpfnZe8V--g align="left")

Kuralların da kontrolünü yaptıktan sonra tarayıcıdan [http://PUBLIC\_IP](http://PUBLIC_IP) adresi ile erişim sağlayabiliriz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXeRa2HhS2rD1OB4_bs-Z0MOnysXi_-WQ7QiPoq3p65eJI_kl0YBsXmCu-4TaonBZCWoqfof-zRYBtL8nfJdgqMqdixtGOw3D8G45iUnsXfKdnL_XiJAt_QxNA34s6qLomJi3WChm1HU8YTCnciY?key=lzOqzu63NCHzpfnZe8V--g align="left")

Sonrasında veritabanı ile ilgili bilgileri girelim.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdksvBpcVvTmrMYevS7kcOC3CWIhaBJuBEoglGjwY41NkpUcKNVuSajluVZEvBdJEQ8VnxUAC5r1378tai9tABkjGQqJ5OKQod_EdJh6S5GhuIg4Lg_acS4gxkmy8Fi0NRQ7f9Ns5eFiCsmBGlK1Yw6WxJ_?key=lzOqzu63NCHzpfnZe8V--g align="left")

Kullanıcı ismini ve bilgilerini de ekledikten sonra giriş ekranı açılıyor.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcGtfsStYkWKD6XwKAel0vgQ9DpGsIJ8-zbXLmrGuXboa2m8GOkSLeJktAP_R5sCicKK7a7hMtgGAmrFscesDSMJSKBv9rvbgUH1ZNYWWVNFKPuM_3tdGe1TMkvkrkZAFNhFBZyPiNXYAHW0rqdIBvYe14?key=lzOqzu63NCHzpfnZe8V--g align="left")

Portala erişim sağlıyoruz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdB546KXt_diBXWTtuYxShHC4X_Bgj9dBW1mR10B4Sx7n_qklDcqTqSWHSTM17ncl5_FzzakAVwceOGbO4Fo-ommhaKavhqOl0rbB9ySRAjUb-SwRP1NIQq07uWK_p0QlpPGfO-SbXb9zdEDuROgpoGMqZ4?key=lzOqzu63NCHzpfnZe8V--g align="left")

Bize özel bir sayfa eklemek istediğimiz zaman /var/www/html dizinine test.html dosyasını ekleyebiliriz. 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcAtBMRdK-3KReJyI--RZLlhNUK0VureahnCjp0A8gir3pXSRboa_Kpwf1tAqsdwbKG81Zop4ZBZdS1ap4-lHKfGlavKY-Wc9EiVIr_tKNzhahayXovMeuOj-cKWRwjW8V6wi2yT1L4b9_KCQTvFQIYcuqq?key=lzOqzu63NCHzpfnZe8V--g align="left")

Ek olarak wordpress kullanıcı bilgilerimizin veritabanında tutulduğunu da görüntüleyebiliriz. Veritabanına bağlanıp, wp\_user tablosunda bir sql sorgusu çalıştırıyoruz. Kullanıcı adı ve parolanın hash bilgisini görebiliriz.

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfJFxjPBofUCxf1bRXdUnsBQLAGGhdb3SsWlPHbd825eN74HrFmv6Y7bSJmTr7yjMCh2GyDhNaC8zcWyQCHi4TSzR2d1JlkAaOUrHIh5xgAPMHsgqYmTbvSBmA-Y0WaE_4Z5D4TtQiq6yQTlBjj1PTZztsU?key=lzOqzu63NCHzpfnZe8V--g align="left")