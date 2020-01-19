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

İlk scriptimizle (Sample 1) başlayalım.






[mytoolbag_dl]: https://github.com/batuhankutluca/Powershell-Decoding-Helper-Tools
[hexeditor_dl]: https://mh-nexus.de/en/downloads.php?product=HxD20
[ida_dl]: https://www.hex-rays.com/products/ida/support/download.shtml
[scripts_dl]: https://www.google.com
[scumbots_twitter]: https://twitter.com/ScumBots

