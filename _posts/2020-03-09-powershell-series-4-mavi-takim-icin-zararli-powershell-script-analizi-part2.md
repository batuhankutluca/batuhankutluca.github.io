---
layout: post
title: "Powershell Serisi -4- [Mavi Takım İçin Zararlı Powershell Script Analizi Part 2]"
feature-img: 'assets/img/powershell-1/bg.png'
date: 2020-03-09 00:00:00
author: batuhankutluca
tags: [Powershell, Obfuscate, Obfuscation, Deobfuscate, Deobfuscation, Enhanced Logging, Logging, Module Logging, Script Block Logging, Transcription]
---

<p align="justify">Herkese selam. Bu yazıda önceki yazının devamı olarak kalan 3 scriptin analizi hakkında bilgi paylaşmaya çalışacağım.</p>

<p align="justify">Aşağıdaki linkten scriptlere ulaşabilirsiniz.</p>

* [Script Repo][scripts_dl] 

## Sample 5

İlk scriptimizle (Sample 5) başlayalım.

<p align="justify">Dosyayı açtığımızda bir powershell kod bloğuyla karşılaşıyoruz. Diğerlerinden farklı olarak burada çalıştırılan kod bloğu komut satırı yerine fonksiyon ve parametreler aracılığıyla çalıştırılmaktadır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/1.png)

<p align="justify">Göz attığımızda kodumuzun hidden yani arayüz oluşturmadan çalıştığını fark ediyoruz. Daha sonra "p152b" adlı fonksiyonu incelemeye başlıyoruz. Fonksiyonun aldığı parametre fonksiyon içinde "$ma865fa" değişkenine eşitleniyor. For döngüsüne baktığımızda "$ma865fa" değişkeninin içeriğinin 2şer okunarak byte'a dönüştürüldüğünü görüyoruz. Yani fonksiyona verilecek parametrenin bir hex stream olacağını anlıyoruz. Hemen aşağısında bir xor işlemi uygulandığını görüyoruz. XOR keyi olarak "$d5726d" değişkeni kullanılıyor yani üst tarafda gördüğümüz "b5ce91" stringi bizim XOR keyimiz oluyor. XOR işleminden sonra dönen her 2li karakterimiz char'a dönüştürülüyor. Buradan da çıktımızın bir string olacağını anlıyoruz. Son olarak fonksiyondan return edilen veri bizim çıktımız olmuş oluyor.</p>

<p align="justify">XOR işlemi için input ve xor keyimizin ne olduğunu anladıktan sonra XORlama işlemine geçiyoruz. Input olarak "$u9558a" değişkeninin değerini (fonksiyona yollanan değer) ve XOR keyimizi ("b5ce91") XOR toolumuzda ilgili yerlere giriyoruz. Input olarak hex, çıktı olarak "string" seçeceğimizi önceden belirlemiştik. Key tipimizin ne olduğunu belirlerken anlamlı bir şey çıkartana kadar seçenekleri denememiz gerekiyor. Powershell'in xor işlemi sırasında key tipini neye göre algıladığı hakkında bir fikrim yok.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/2.png)

<p align="justify">Anlamlı verimizi string key seçeneğiyle ortaya çıkarmış olduk.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/3.png)

<p align="justify">Çıktımızı biraz daha okunur hale getirdiğimizde bir WebClient objesi oluşturulup dosya indirildiğini, daha sonra indirilen dosyanın çalıştırıldığını görüyoruz. İstek atılan url bilgisi yine xorlanmış bir şekilde karşımıza çıkıyor. Analizimize bağlantı kurulan urli belirlemeye çalışarak devam ediyoruz. XOR işlemi olarak kullanılan fonksiyonun "p152b" olduğunu scriptin en altında görüyoruz. Key olarak yine önceki işlemdeki key kullanılmış.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/4.png)

<p align="justify">Ek olarak indirilen dosyanın "Appdata" lokasyonuna "u2219e5" ismiyle .exe uzantılı bir şekilde kaydedildiğini görüyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/5.png)

![img]({{ site.baseurl }}/assets/img/powershell-4/6.png)

## Sample 6

<p align="justify">Dosyayı açtığımızda base64 encodelanmış verimizi görüyoruz ve analizimize decode ederek başlıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/7.png)

<p align="justify">Decode edilmiş verimizi incelediğimizde önce base64 encodelanmış, daha sonra GZiplenmiş bir dizi veriyle karşılaşıyoruz. Hatırlayacak olursak bir önceki yazıdaki script bloğuyla hemen hemen aynı yapıya sahiptir. Aynı yöntemi uygulayarak decompress işlemimizi gerçekleştiriyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/8.png)

<p align="justify">Decompress işlemimizi gerçekleştirip çıktımızı incelemeye başlıyoruz. Klasik bir meterpreter kodu yapısıyla karşı karşıyayız. Kodun aşağısında base64 encodelanmış shellcodeumuzu görüyoruz ve hex streame dönüştürmek için "Base64EncodeDecode" toolumuzu kullanıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/9.png)

<p align="justify">Shellcode'a ulaştığımızı FC E8 magic bytelardan anlayabiliriz. Hex streamimizi herhangi bir hex editör yardımıyla açıyoruz. Windows Defender kapalı bir sistemde çalışmanızı öneririm çünkü çoğu zaman Defender dosyayı kaydetmenize engel olacaktır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/10.png)

<p align="justify">Cleartext bir IP/Domain/Port bilgisi elde edemedik. Dolayısıyla analizimize binarymizi disassemble ederek devam etmek zorundayız. Binarymizi disassembler aracılığıyla açıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/11.png)

Bir önceki yazıda disassemble edilen veride art arda 2 adet push komutu aramıştık. Yine aynı şekilde aradığımızda bu sefer karşımıza iki adet sonuç çıktı. Hangisi olduğunu anlamak için hex veri içinde 0200 aramak yeterli olacaktır. Buradaki 0200 hex verisi, [winsocket apisinin][msdn] AF_INET parametresine denk gelmektedir. 02 verisi bize kullanılacak IP protokolünün IPv4 olduğunu söylemektedir. Dolayısıyla decimale çevireceğimiz kısım altta gözüken push fonksiyonları olmalıdır. Decimale çevirirken hex streamini Little-Endian şeklinde almamız gerektiğini unutmayalım. 

![img]({{ site.baseurl }}/assets/img/powershell-4/12.png)

![img]({{ site.baseurl }}/assets/img/powershell-4/13.png)

<p align="justify">Verimizi doğru şekilde decimale çevirdikten sonra C2 bilgimizi 76.218.94.80 ve connection kurulan portu 4444 olarak elde ettik.</p>

## Sample 7

<p align="justify">Son script olarak Invoke-Obfuscation modülünün içinde gelen "Special Characters" methoduyla obfuscate edilmiş bir scripti nasıl deobfuscate edebileceğimizi anlatmaya çalışacağım. Örnek olması açısından bir script oluşturdum.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/14.png)

<p align="justify">Scripte ilk baktığımızda okunabilir hiçbir şey göremeyeceğiz. Bu yüzden yapacağımız ilk şey delimiter olarak kullanılan ";" 'e göre kod bloğumuzu biraz daha okunabilir hale getirmek olacaktır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/15.png)

<p align="justify">Sonra ilk kısmın ne işe yaradığını anlamaya çalışalım. Kodu Powershell ISE içine atıp çalıştırıyorum ve her değişkenin değerini işlem sonrasında ekrana bastırıyorum.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/16.png)

<p align="justify">Buradan çıkarabileceğimiz sonuç her değişkene 0'dan 9'a kadar bir değer atıyor. O halde script içinde ilgili değişkenleri, değerleriyle replace edebiliriz. </p>

![img]({{ site.baseurl }}/assets/img/powershell-4/17.png)

<p align="justify">Replace işleminden sonra bir alt satıra geçiyoruz. "$(@{})" değişkeninin neye karşılık geldiğini anlamak için yine aynı işlemi gerçekleştiriyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/18.png)

<p align="justify">"$(@{})" değişkenimizin "System.Collections.Hashtable" stringine denk geldiğini gördük. [7], [19] gibi değerler bu stringimizin ilgili indexindeki harfe denk geliyor yani bir array gibi davranıp ilgili indexteki karaktere eşleniyor. Bunu anladıktan sonra "${~%}" değişkenimizin neye karşılık geldiğini çözebiliriz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/19.png)

<p align="justify">Sırasıyla C,h ve a karakterlerine eşlendiğini gördük. Ancak aynı satırın sonunda bulunan "$?"[1] dğeişkeninin değerini bilmiyoruz. Aynı yöntemle bulmaya çalışalım.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/20.png)

<p align="justify">Dönen cevap "True" oldu ve 1. indexteki karaktere eşlediğimizde "r" harfine denk geldiğini anlıyoruz. Yani "${~%}" değişkenimiz [Char] değerine karşılık geliyor. Yine "${~%}" gördüğümüz yeri [Char] ile değiştirerek devam ediyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/21.png)

<p align="justify">Yapmamız gereken son şey ilgili char'ların neye karşılık geldiğini bulmak olacaktır. Bunun için "CharCodeConverter" toolumuzu kullanabiliriz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/22.png)

<p align="justify">İlk başta gördüğümüz ve anlamlandıramadığımız karmaşık kodun aslında ne çalıştırdığını görmüş olduk. :)</p>

<p align="justify">Yanlış veya eksik bir bilgi söz konusuysa lütfen sayfanın en aşağısında bulunan sosyal medya linklerim üzerinden beni bilgilendirin. Bir sonraki yazıda görüşmek üzere. </p>










[msdn]: https://docs.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-socket
[scripts_dl]: https://github.com/batuhankutluca/Malicious-Powershell-Samples