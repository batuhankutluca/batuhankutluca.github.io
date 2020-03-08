---
layout: post
title: "Powershell Serisi -4- [Mavi Takım İçin Zararlı Powershell Script Analizi Part 2]"
feature-img: 'assets/img/powershell-1/bg.png'
date: 2020-03-09 00:00:00
author: batuhankutluca
tags: [Powershell, Obfuscate, Obfuscation, Deobfuscate, Deobfuscation, Enhanced Logging, Logging, Module Logging, Script Block Logging, Transcription]
---

<p align="justify">Herkese selam. Bu yazıda önceki yazının devamı olarak kalan 3 scriptin analizi hakkında bilgi paylaşmaya çalışacağım.</p> 

## Sample 1

İlk scriptimizle (Sample 1) başlayalım.

<p align="justify">Dosyayı açtığımızda bir powershell kod bloğuyla karşılaşıyoruz. Diğerlerinden farklı olarak burada çalıştırılan kod bloğu parametreler aracılığıyla çalıştırılmaktadır.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/1.png)

<p align="justify">Göz attığımızda kodumuzun hidden yani arayüz oluşturmadan çalıştığını fark ediyoruz. Daha sonra "p152b" adlı fonksiyonu incelemeye başlıyoruz. Fonksiyonun aldığı parametre fonksiyon içinde "$ma865fa" değişkenine eşitleniyor. For döngüsüne baktığımızda "$ma865fa" değişkeninin içeriğinin 2şer okunarak byte'a dönüştürüldüğünü görüyoruz. Yani fonksiyona verilecek parametrenin bir hex stream olacağını anlıyoruz. Hemen aşağısında bir xor işlemi uygulandığını görüyoruz. XOR keyi olarak "$d5726d" değişkeni kullanılıyor yani üst tarafda gördüğümüz "b5ce91" stringi bizim XOR keyimiz oluyor. XOR işleminden sonra dönen her 2li karakterimiz char'a dönüştürülüyor. Buradan da çıktımızın bir string olacağını anlıyoruz. Son olarak fonksiyondan return edilen veri bizim çıktımız olmuş oluyor.</p>

<p align="justify">XOR işlemi için input ve xor keyimizin ne olduğunu anladıktan sonra XORlama işlemine geçiyoruz. Input olarak "$u9558a" değişkeninin değerini (fonksiyona yollanan değer) ve XOR keyimizi ("b5ce91") XOR toolumuzda ilgili yerlere giriyoruz. Input olarak hex, çıktı olarak "string" seçeceğimizi önceden belirlemiştik. Key tipimizin ne olduğunu belirlerken anlamlı bir şey çıkartana kadar seçenekleri denememiz gerekiyor. Powershell'in xor işlemi sırasında key tipini neye göre algıladığı hakkında bir fikrim yok.</p>

![img]({{ site.baseurl }}/assets/img/powershell-4/2.png)
