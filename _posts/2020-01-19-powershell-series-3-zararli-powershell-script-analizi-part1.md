---
layout: post
title: "Powershell Serisi -3- [Zararlı Powershell Script Analizi Part 1]"
feature-img: 'assets/img/powershell-1/bg.png'
date: 2020-01-19 00:00:00
author: batuhankutluca
tags: [Powershell, Obfuscate, Obfuscation, Deobfuscate, Deobfuscation, Enhanced Logging, Logging, Module Logging, Script Block Logging, Transcription]
---

<p align="justify">Herkese selam. Bu yazıda sizleri zararlı powershell scriptlerini analiz etme hakkında elimden geldiğince aydınlatmaya çalışacağım. Analizin ana odağı zararlı scriptlerden çalışması hedeflenen shellcode'a ulaşıp IP/Domain ve port bilgisi elde etmek olacaktır. Analizin yanı sıra oluşturduğum yardımcı toolların da nasıl kullanılacağını anlatmaya çalışacağım.</p> 

<p align="justify">Analiz yaparken yazdığım toollar haricinde hex editore ve bir adet disassemblera ihtiyacımız olacak. Ben hex editor olarak HxD ve disassembler olarak IDA kullanıyorum ve yazının devamındaki görseller bu uygulamalara ait olacaktır. Ancak mantığını oturttuktan sonra hangi toolu kullandığınızın pek bir önemi olacağını düşünmüyorum. Toolların ve analizini yapacağımız scriptlerin linklerine aşağıdan ulaşabilirsiniz.</p>

* [My Toolbag][mytoolbag_dl]
* [HxD Hex Editor][hexeditor_dl]
* [IDA Disassembler][ida_dl]
* [Script Repo][scripts_dl]

Örnek scriptlerin çoğunu [Scumbots][scumbots_twitter] twitter hesabının paylaşımlarından aldım. Bu twitter hesabı pastebin.com'da paylaşılan binary/powershell kodlarını otomatik olarak analiz edip IOC tanımlaması yapıyor. Çeşitlilik açısından güzel bir repo olduğunu düşünüyorum.

<p align="justify">Toplamda 8 adet scriptin analizini anlatacağım ancak ilk dördünü bu yazıda paylaşacağım. Kalan dördünü aynı yazının Part2 olarak gelen devam yazısında anlatmaya çalışacağım. Yazdığım toollar scripti çalıştırmadan decoding işlemi yaptığı için tolların kullanımı sırasında herhangi bir risk oluşmayacaktır ancak scriptler Defender tarafından karantinaya alındığı için Test VM'inizde Defender'ı kapatmanızı öneririm (veya her bir dosya için cihazda izin ver seçeneğini kullanabilirsiniz).</p>

## Sample 1

İlk scriptimizle (Sample 1) başlayalım.

<p align="justify">Dosyayı açtığımızda base64 encodelanmış bir veriyle karşılaşıyoruz. "Base64EncodeDecode.exe" toolumuzla veriyi decode diyoruz. Veriyi doğru encoding type'ı bulana kadar decode etmeye çalışıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/1.png)

<p align="justify">Başarılı bir şekilde decode ettikten sonra script bloğunun "hxxps://5.top4top.net/p_1372hc5jv1.jpg" dosyasının içeriğini okuyup "defender" adlı fonksiyonu çağırdığını görüyoruz. "p_1372hc5jv1.jpg" bir resim olarak gözükse de dosyasının içeriğini herhangi bir text editor aracılığıyla okuduğumuzda "defender" adlı fonksiyonla ve "$payload" değişkenine atanmış bir dizi decimal veriyle karşılaşıyoruz. Ayrıca "Reflection.Assembly" fonksiyonundan payloadın memory üzerinde çalışacağını anlıyoruz. Bu yöntem genel olarak "fileless malware" türündeki zararlılarda görülmektedir. .NET direk olarak memory üzerinde binary çalıştırmaya yarayan methodlara sahiptir ve bu özellik sayesinde diske herhangi bir dosya yazmadan memory üzerinde bir alan allocate edip bu alan üzerinde binary çalıştırılabilir. Powershell de .NET tabanlı olduğu için bu tarz özellikler zararlı amaçlarla da kullanılabilmektedir. Çoğu zaman AV/EDR bypass için bu yöntem kullanılmaktadır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/2.png)

<p align="justify">$payload değişkeni içindeki decimal verinin çalıştırılması amaçlandığından bu veriyi hex formatına dönüştürüp analizimize devam etmemiz gerekiyor. </p>

[Bu linkten][rapid] decimal verimizi hex formatına dönüştürüyoruz.

![img]({{ site.baseurl }}/assets/img/powershell-3/3.png)

Magic Byte'ları kontrol ettiğimizde "4D 5A" ile başladığını (MZ Header) görüyoruz ve bunun bir .exe uzantılı dosya olduğunu anlıyoruz. Exe analizi konusunda çok derin bilgim olmadığı için dosyayı [any.run][anyrun1] üzerinden analize verdim.

![img]({{ site.baseurl }}/assets/img/powershell-3/4.png)

Karşı sunucu ayakta olmadığı için herhangi bir data akışı sağlanmadı ancak dosya sha'sını [VirusTotal][vt1] üzerinde arattığımızda IOC'lerden dosyanın SamSam ransomware'ına ait bir sample olduğunu anlıyoruz.

![img]({{ site.baseurl }}/assets/img/powershell-3/5.png)

## Sample 2

<p align="justify">Dosyayı açtığımızda bir dizi parametre ve base64 encodelanmış veriyle karşılaşıyoruz. Parametreler kullanıldığı için büyük ihtimalle cmd aracılığıyla çalıştırılan bir kod bloğu olduğunu anlıyoruz. "-w" parametresi ve "1" argümanı powershellimizi "-WindowStyle" seçeneklerinden 1.si yani "Hidden" seçeneğiyle çalıştıracaktır. Bu da powershellin arka planda yani ekrana bir çıktı vermeden çalıştırılacağı anlamına gelmektedir. "/C" parametresi ise "-command" yani devamında gelen çift tırnak içine yazılmış argümanı çalıştır demek oluyor. Çalıştırılacak kodumuza baktığımızda 2 adet alias kullanıldığını fark ediyoruz.</p> 

"sv" aliasını [araştırdığımızda][alias_resource] "Set-Variable", "gv" aliasını araştırdığımızda ise "Get-Variable" fonksiyonlarına karşılık geldiğini görüyoruz.

<p align="justify">Önce "sv" aliası ile "ww" değişkenine "-" değeri atanmış. Yine aynı yöntemle "qC" değişkenine "ec" değeri atanmış. Daha sonra "Lf" değişkeni oluşturulup "gv" aliası ile önce "ww" değişkeninin değeri, sonra "qC" değişkeninin değeri string şeklinde birleştirilip "Lf" değişkenine atanmış. Sonuca bakacak olursak "-ec" değeri "Lf" değişkenine atanmış oluyor. "-ec" parametresi "-EncodedCommand" parametresine karşılık gelmektedir. Yani bir encodelanmış veri çalıştırılacağını anlıyoruz. Burada bir obfuscation işlemi yapılmış. </p> 

![img]({{ site.baseurl }}/assets/img/powershell-3/6.png)

<p align="justify">Analize base64 verimizi decode etmeye çalışarak devam ediyoruz. Toolumuzla direkt olarak decode etmeye çalışırsak hata alacağız. Çünkü içerisinde "+" karakteri kullanılmış yani iki parça kod string olarak birleştirilmiş. Bu yüzden '+' kısmını silmemiz gerekiyor.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/7.png)

<p align="justify">Veriyi düzgün hale getirdikten sonra doğru encoding type'ı bulana kadar decode etmeye çalışıyoruz ve "Unicode" olarak başarılı bir şekilde decode edebiliyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/8.png)

<p align="justify">Decode edilmiş veriye göz attığımızda bir hex stream gözümüze çarpıyor. Daha da önemlisi "fc, e8, 82" magic bytelarına sahip olmasından dolayı bunun bir meterpreter shellcode olduğunu anlıyoruz. Eğer msfvenom ile herhangi bir payload oluşturup hex editor yardımıyla incelerseniz payloadın "fc, e8, 82" magic bytelarına sahip olduğunu göreceksiniz. Daha sonra analizimize bu shellcode aracılığıyla bağlantı kurulmaya çalışılan C2 IP/Domain ve port bilgisini elde etmeye çalışıyoruz. Text editor aracılığıyla :, karakterlerini silip hex editorümüze kopyalıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/9.png)

<p align="justify">Decoded text bölümünde en altta "192.168.43.179" IP'sini görmekteyiz. Ayrıca /1IUpVtAgm... ile başlayan cleartext kısım bir dizini ifade etmektedir. Yani connection 192.168.43.179/1IUpVtAgm... web sunucu dizinine sağlanmaya çalışılmaktadır. Bu IP extraction her zaman bu şekilde cleartext olarak karşımıza çıkmamaktadır. Genelde bir dizine connection atıldığında bu IP-Domain/Path bilgisini cleartext olarak görebiliyoruz. Bu tip durumlarda path üzerinden tarayıcı zafiyetini sömürmeye yarayan zararlı kod yerleştirilmiş web sayfaları servis ediliyor olabilir.</p>

<p align="justify">Ek olarak hex editor aracılığıyla shellcode'umuzu dosya olarak kaydedip disassembler yardımıyla stringleri çıkartabiliriz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/10.png)

<p align="justify">Buradan yine path bilgisini görebiliyoruz.</p>

!! 77696E6954684C7726 wininetx86!!

## Sample 3

<p align="justify">Dosyayı açtığımızda bir char array ve en sonda xor işlemi için bir key görüyoruz. En başta ise "((vaRIABLe '*MDr*').naME[3,11,2]-JOIn'')" ifadesi yer alıyor. Bu ifadenin ne anlama geldiğini anlamak için powershell açıp komutu çalıştırıyoruz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/11.png)

<p align="justify">Dosyayı açtığımızda bir char array ve en sonda xor işlemi için bir key görüyoruz. En başta ise "((vaRIABLe '*MDr*').naME[3,11,2]-JOIn'')" ifadesi yer alıyor. Bu ifadenin ne anlama geldiğini anlamak için powershell açıp komutu çalıştırıyoruz. Karşımıza "iex" komutu yani InvokeExpression fonksiyonu çıkıyor. Bu fonksiyon kendinden sonra gelen bir dizi komutu çalıştırmaya yarıyor. Daha detaylı görmek için "variable *mdr*" komutunu çalıştırıyorum. "variable" komutunu bir üstteki gv aliası gibi düşünebiliriz yani bir değişkenin değerini almak için kullanılıyor. Sırasıyla arrayin 3,11 ve 2. elemanlarını aldığımızda i-e-x değerleriyle karşılaşıyoruz ve join komutuyla birleştirildiğinde InvokeExpression ifadesi karşımıza gelmiş oluyor. Bu kalıp çoğu obfuscate edilmiş scriptlerde karşımıza çıkmaktadır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-3/12.png)

Burayı anladıktan sonra kalan işlem char arrayi önce string haline getirip daha sonra xorlamaktır. Son olarak bir dizi powershell komutuyla karşılaşacağız ve bu komutların çalışması için hepsinin başına IEX yani InvokeExpression fonksiyonuna ihtiyacımız var. Dosyamızın en sonuna baktığımızda xor işlemi için belirtilen key "1D" olarak gözüküyor. "CharCodeConverter.exe" toolumuzu açıp charcode bölümünü input olarak giriyoruz.</p>









[mytoolbag_dl]: https://github.com/batuhankutluca/Powershell-Decoding-Helper-Tools
[hexeditor_dl]: https://mh-nexus.de/en/downloads.php?product=HxD20
[ida_dl]: https://www.hex-rays.com/products/ida/support/download.shtml
[scripts_dl]: https://www.google.com
[scumbots_twitter]: https://twitter.com/ScumBots
[rapid]: https://www.rapidtables.com/convert/number/ascii-hex-bin-dec-converter.html
[anyrun1]: https://app.any.run/tasks/48b5012b-4987-4925-ac00-9677c100615a/
[vt1]: https://www.virustotal.com/gui/file/626879e64f571e21902bdc2f249ce247e03420e8656990d54f3ab4ceb99b4fb4/detection
[alias_resource]: https://ilovepowershell.com/2011/11/03/list-of-top-powershell-alias/


