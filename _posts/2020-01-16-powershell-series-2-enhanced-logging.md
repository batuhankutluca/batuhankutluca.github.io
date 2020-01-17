---
layout: post
title: "Powershell Serisi -2- [Enhanced Logging]"
feature-img: 'assets/img/powershell-1/rsz_ps.png'
date: 2020-01-16 00:00:00
author: batuhankutluca
tags: [Powershell, Obfuscate, Obfuscation, Deobfuscate, Deobfuscation, Enhanced Logging, Logging, Module Logging, Script Block Logging, Transcription]
---

<p align="justify">Herkese selam. Bu yazıda sizleri enhanced powershell logging hakkında elimden geldiğince aydınlatmaya çalışacağım. </p>

<p align="justify">Powershell, default olarak Windows'un her sürümünde aktif olarak gelmektedir. Son sürümü 7 olmasına rağmen sıfır kurulumda 5 sürümü yüklü olarak karşımıza çıkar. Sisteminizde hangi powershell sürümünün yüklü olduğunu powershell veya powershell ISE komut satırına "$Host.Version" yazıp "Major" fieldından öğrenebilirsiniz. </p> 

![img]({{ site.baseurl }}/assets/img/powershell-2/1.png)

<p align="justify"> 5 Sürümünün önceki sürümlerden en büyük farkı, içinde barındırdığı detaylı log üretme mekanizmasıdır ki bu özellik sayesinde çalışan script bloğunun tamamını, çalışan kod her ne kadar obfuscated olursa olsun en son çalışan de-obfuscated kodu ve çalışan kodun çıktısını loglama imkanını bize sunar. </p>

# Module Logging

<p align="justify">Bahsettiğim detaylı logginglerin ilkiyle yani "Module Logging" özelliğini açarak ne işe yaradığını gözlemleyelim. -Test ortamını İngilizce Windows kurduğum için menüler İngilizce olacaktır- Bütün logging özelliklerini GPO üzerinden aktif edeceğiz. Bende domain ortamı olmadığı için VM'imde "Local Group Policy Editor" üzerinden aktif edeceğim. LGPO'ya çalıştıra "gpedit.msc" yazarak erişebiliriz.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/2.png)

<p align="justify">Önümüze gelen pencerede 2 adet bölüm yer almaktadır. "Computer Configuration" altında yaptığınız değişiklikler o cihazdaki bütün kullanıcılar için geçerli olurken "User Configuration" altında yaptığınız değişiklikler sadece düzenleme yaptığınız kullanıcı seviyesinde geçerli olacaktır. Powershell'le ilgili logging özelliklerine Administrative Templates -> Windows Components -> Windows Powershell sekmelerini takip ederek ulaşabiliriz. </p>

![img]({{ site.baseurl }}/assets/img/powershell-2/3.png)

<p align="justify">Bütün özellikler default olarak kapalı durumda geliyor. Açmak istediğimiz özelliğin üstüne çift tıklıyoruz. Karşımıza çıkan pencerede durumu "Enabled" konumuna getiriyoruz. </p>

![img]({{ site.baseurl }}/assets/img/powershell-2/4.png)

<p align="justify">Help kısmında özelliğin ne işe yaradığı hakkında kısa bir bilgi bulunmaktadır. Özetleyecek olursak powershell üzerinde barınan modülleri (modülden kasıt komut satırına yazdığınız her türlü powershell fonksiyonu "Write-Host, Ipv4Address" gibi o fonksiyonun ait olduğu modüllerdir.</p> 
[MSDN][ref] linkinden hangi fonksiyonun hangi modüle ait olduğunu görebilirsiniz.) ve yanında kullanılan argümanları loglamayı sağlar. "Module Names" bölümünden hangi modüller için log üretileceğini belirlememiz gerekiyor. "Value" bölümüne "*" verdiğiniz takdirde bütün modüller loglanacaktır ancak bir domain ortamında bütün modüllerin loglanması ve SIEM ortamına aktarılması, storage bakımından problem teşkil edebilir.

![img]({{ site.baseurl }}/assets/img/powershell-2/5.png)

<p align="justify"> Örnek olması açısından "Write-Host" fonksiyonunun ait olduğu "Microsoft.PowerShell.Utility" modülünün loglanmasını istiyorum.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/6.png)

<p align="justify">Yaptığımız değişikliklerin uygulanması için powershell veya cmd komut satırına "gpupdate /force" komutunu yazıyorum. Değişiklik başarıyla uygulandıktan sonra böyle bir çıktı veriyor. </p>

![img]({{ site.baseurl }}/assets/img/powershell-2/7.png)

<p align="justify">Policy güncellendikten sonra mevcut powershell.exe'yi kapatıp yeni bir powershell.exe açmak zorundayım çünkü yeni açacağım powershell yeni policy uygulanmış şekilde açılacaktır. Test amaçlı Write-Host "Batuhan" yazıyorum ve oluşan logu inceliyorum.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/8.png)

<p align="justify">Bunun yanı sıra logumuzun oluştuğunu Event Viewer sayesinde göreceğiz. Oluşacak logun EventID'si 800'dür. İlgili Powershell Loguna Application and Services Logs -> Windows PowerShell sekmelerinden ulaşacağız.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/9.png)

<p align="justify">Ekran görüntüsünde görüldüğü üzere loglamak istediğimiz bir modülün fonksiyonu kullanıldığında bu olayı başarılı bir şekilde loglayabildik. "CommandInvocation" kısmında kullanılan fonksiyonu, "ParameterBinding" kısmında ise kullanılan argümanı görebiliyoruz.</p>

<p align="justify">Ekstra olarak powershell script bloğu komut satırından değil de bir .ps1 dosyası tarafından çalıştırılsaydı bunu da Script fieldından görecektik.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/10.png)

![img]({{ site.baseurl }}/assets/img/powershell-2/11.png)

# Script Block Logging

<p align="justify">Bu özellik powershell tarafından çalıştırılan komut veya komut dosyaları(.ps1) farketmeksizin tüm içeriği kaydeder. Çalıştırılan komut obfuscated olsa dahi en son çalıştırılan de-obfuscate olmuş kod loglanır. Eğer kod bir şekilde AMSI(Anti-Malware Scan Engine) tarafından bloklanırsa çalışmayan kod bloğu da loglanır. Bu logun Event ID'si 4104'tür. Ek olarak "Log script block execution start / stop events" kutucuğu işaretlendiğinde, script bloğunun ne zaman çalışmaya başlayıp ne zaman sonlandığı hakkında bilgi veren 2 adet farklı log üretilir. 4105 scriptin çalışmaya başladığı zamanı, 4016 ne zaman sonlandığını belirtir.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/12.png)

<p align="justify">Test için bir .ps1 dosyası oluşturalım. İçi görseldeki gibi birden fazla komut içersin.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/13.png)

<p align="justify">Yine cmd aracılığıyla dosyamızı çalıştıralım. Hemen ardınan EventViewer'dan oluşan loglara bakalım.</p>

<p align="justify">Script block loggingi özelliğinin oluşturduğu loglar Application and Service Logs -> Microsoft -> Windows -> Powershell -> Operational sekmelerinin altında bulunmaktadır. </p>

![img]({{ site.baseurl }}/assets/img/powershell-2/14.png)

<p align="justify">4104 EventID'li oluşan loga baktığımızda .ps1 dosyamızın içindeki komutları harfi harfine logladığını gördük. Ayrıca start/stop events kutucuğunu da işaretlediğimiz için komutların ne zaman çalışmaya başlayıp sonlandığını belirten 4105 ve 4016 EventID'lerinin de oluştuğunu gördük. </p>

![img]({{ site.baseurl }}/assets/img/powershell-2/15.png)

<p align="justify">Son olarak aynı 2 komutu obfuscate edip oluşturduğu logu inceleyelim. Obfuscate işlemi için Invoke-Obfuscation modülünü kullanacağım. En son çıktımı alıp ps dosyamın içine yazıyorum.</p>

<p align="justify">Son çıktı şu şekilde oldu.
[STrINg]::joIn('' ,(( 26,63,36 , 57, 40,96 ,5, 34 ,62 , 57 ,109 ,111,15 ,44, 57 , 56, 37,44,35 , 111,118 ,109,36 , 61 , 46 , 34 , 35 ,43, 36,42) |FOReACh {[CHAR]($_-BXOR '0x4D') } ))| & ( $Env:COMSPEc[4,24,25]-joIN'')</p>

<p align="justify">Yine cmd aracılığıyla ps1 dosyamı çalıştırıyorum ve oluşan loglara bakıyorum.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/16.png)

![img]({{ site.baseurl }}/assets/img/powershell-2/17.png)

<p align="justify">Ben script bloğumu 1 kere obfuscate etmiştim ancak 50 kere de obfuscate etseydim yine bir şey değişmeyecekti çünkü powershell obfuscate edilmiş kod bloğunu çalıştırmak için önce kendi de deobfuscate işlemi gerçekleştiriyor ve kodun en sade halini logluyor.</p>

# Transcription

<p align="justify">Bu özellik çalıştırılan script bloğunun ne çıktı verdiğini loglamaya yarar. Seçtiğimiz bir dizinde timestampli bir şekilde text dosyasına yazar.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/18.png)

<p align="justify">Yine cmd aracılığıyla dosyamızı çalıştırıp belirttiğimiz dizini kontrol edelim.</p>

![img]({{ site.baseurl }}/assets/img/powershell-2/19.png)

![img]({{ site.baseurl }}/assets/img/powershell-2/20.png)

<p align="justify">Görüldüğü üzere çalışan script bloğunun ne çıktı verdiğini metadatayla birlikte başarıyla loglayabildik.</p>

<p align="justify">Yazdığım her şey çeşitli kaynaklardan edindiğim bilgilerin ve araştırmalarımın sonuçlarının kendimce yorumlanmış halidir. Yanlış veya eksik bir bilgi söz konusuysa lütfen sayfanın en aşağısında bulunan sosyal medya linklerim üzerinden beni bilgilendirin. Bir sonraki yazıda görüşmek üzere. </p>

[ref]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/write-host?view=powershell-5.1