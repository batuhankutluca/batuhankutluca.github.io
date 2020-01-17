---
layout: post
title: "Powershell Serisi -1- [Encoding Tipleri]"
feature-img: 'assets/img/powershell-1/bg.jpg'
date: 2020-01-12 19:00:00
author: batuhankutluca
tags: [Powershell, Obfuscate, Obfuscation, Unicode, ASCII, UTF8, Decode, Encode]
---

<p align="justify">Uzun bir aradan sonra tekrardan herkese selam. İlk yazımı paylaştığımda amacım aktif bir blog sayfası oluşturmaktı ancak üşengeçliğim ve çeşitli sebeplerden ötürü herhangi bir içerik üretip sitemi aktif tutamadım. Araştırdığım ve öğrendiğim yeni bilgilerle burayı daha da aktif tutmayı planlıyorum. Bu ve devamındaki birkaç yazının içeriği genel olarak Powershell, Powershell Obfuscation - Deobfuscation, Zararlı Powershell Script Analizi, Powershell Enhanced Logging, Powershell Scriptlerinden C2 IP ve port bilgisi elde etme hakkında olacaktır.</p>

<p align="justify">Bu yazı üstte bahsettiğim konu başlıklarına ithafen konulara giriş niteliğinde olacaktır. Zararlı ve obfuscate edilmiş bir Powershell Scriptini anlamak ve hızlı bir şekilde Incident-Response yapabilmek için bir süredir araştırma yapıp kendimce toollar geliştiriyordum. Buradaki hızlı Incident-Response'dan kastım, çalışan scriptin derinine inerek içerdiği shellcode'un bağlantı kurmaya çalıştığı IP ve Port bilgisini en hızlı şekilde elde etmektir. Toolbagimi oluşturmak için belli başlı konu başlıkları hakkında biraz araştırma yapmam gerekti.</p>

Pentest sırasında tespit ettiğiniz veya herhangi bir Sandbox'tan indirdiğiniz Scripti online toollarla deobfuscate etmeniz çoğu zaman mümkün olacaktır [CyberChef gibi][cyber-chef] ancak olayların arka planda nasıl geliştiğini anlamak ve kendi toolbaginizi oluşturmak istiyorsanız Encoding, Charset, Charcode, Unicode, ASCII, UTF-8 gibi konu başlıkları hakkında araştırma yapmanız gerekmektedir. [Linkteki][uni] yazı konu başlıkları hakkında faydalı bilgiler sunmaktadır. Ben sadece özet bir bilgi vermeye çalışacağım.

<p align="justify">Olaylara kronolojik olarak bakacak olursak bilgisayar ekranında okuduğumuz her harfi alfabemizle eşleştirip anlamlandırırız. Bilgisayarda da her karakter arkada 0 ve 1'lerden oluşmaktadır. Biz "a" yazdığımızda aslında o karakter arkada "01100001" şeklinde tutulur ancak bunu ASCII tablosunda binarynin decimal halini 97'ye yani "a" karakterine eşlemesi sonucunda yapar. Bu karakter eşlemesini yaptığı tabloya "Charset" diyebiliriz. İlk oluşturulan charset ASCII'dir. ASCII tablosunu incelediğinizde sadece İngilizce harfleri göreceksiniz çünkü ASCII Amerikan standartlarına göre üretilmiş bir karakter setidir. Zamanla farklı ülkelerdeki farklı işlemci üreticileri kendi dillerindeki karakterleri kullanabilmek için farklı farklı karakter setleri üretmişlerdir. Kısacası o zamanlarda bunun standart bir hali yoktu. Yani siz japonca bir websiteyi ASCII encoding tipinde görüntülemek istediğinizde karakterler eşlenmeyeceğinden görüntülediğiniz site saçma sapan karakterlerle dolup taşacaktı. Bunu standartlaştırmak için "Unicode" adı verilen bir charset oluşturulmuştur. Bunu dünya üzerinde bütün alfabelerdeki bütün harflerin içine toplandığı bir havuz olarak düşünebilirsiniz. İnternetin yaygınlaşması sonucu bu probleme bir çözüm niteliğinde üretilmiştir.</p>

<p align="justify">Bilgisayar ASCII'de "a" harfini ASCII tablosunda decimal olarak "97" ile eşleştirdiği gibi Unicode'da da U+0097 şeklinde eşleştirir. ASCII için ilk 127 karakter neyse Unicode'da da aynı sırada tutulur. Bir de bu "U+0097" karakterin nasıl verimli bir şekilde saklanacağı problemi vardır. Bunun için de "Encoding" yani o binary'nin nasıl depolanacağını belirten bir sistem geliştirilmiştir. Bunlardan en çok kullanılanları UTF-8 ve UTF-16'dır. Browser'ınız bir websiteyi düzgün görüntüleyebilmesi için hangi encoding tipiyle encode'landığını bilmesi gerekir. HTML5'in default karakter seti UTF-8'dir. Bunu HTTP Response Header bilgisinde görebiliriz. Web sunucu bu bilgiyi bize verir ki browser ona göre web siteyi doğru bir şekilde görüntüleyebilmemize olanak sağlar.</p>

<p align="justify">UTF-8 çoğu yerde charset şeklinde belirtilmektedir ancak terimleri doğru bir şekilde kullanmak istersek - araştırmalarıma göre - bir encoding type olduğunu söyleyebilirim. Ve yine araştırmalarıma dayanarak UTF-8'in latin alfabesi için, UTF-16'nın non-latin alfabeleri için en verimli şekilde saklandığını söyleyebilirim. Birbirlerinden farkı her karakterin kaç byte şeklinde saklandığıdır. Ancak günümüzde kullandığımız bilgisayarların memory problemi olmadığı için nasıl saklandığının pek de bir önemi yoktur diyebiliriz. İşin özeti aslında elimizdeki bir stringi veya arka planda tutulduğu byte halini tek başına bilmek pek bir işimize yaramaz. Çünkü onun aslında ne ifade ettiğini doğru şekilde anlamak için hangi encoding tipiyle ifade edildiğini bilmemiz gerekir. İşte bu noktada charset ve encoding type kavramları büyük bir önem taşımaktadır.</p>

<p align="justify">Olayı örneklendirecek olursak "ァ" bu karakteri doğru şekilde görüntüleyebiliyorsunuz çünkü websunucum size bu karakterin UTF-8 şeklinde encode'landığı bilgisini verdi. Peki biz bunu bir dosyaya kaydedip encode tipini değiştirirsek ne elde ederiz? Notepad'i açıp karakteri yapıştırdım ve kaydet diyip kodlamayı ANSI olarak bıraktım.</p>

![img]({{ site.baseurl }}/assets/img/powershell-1/1.png)

<p>ANSI formatını ASCII'nin biraz daha geniş hali olarak düşünebilirsiniz. Kaydet'e bastığımda karşımıza bir uyarı geldi. Özetle bu karakter için ANSI formatında bir eşleme yoktur, devam ederseniz karakteri doğru görüntüleyemeyeceksiniz şeklinde bir uyarı veriyor. </p> 

![img]({{ site.baseurl }}/assets/img/powershell-1/2.png)

Devam edip dosyayı görüntülemeye kalkarsak "?" şeklinde bir içerikle karşılaşacağız. 

![img]({{ site.baseurl }}/assets/img/powershell-1/3.png)

<p>Ancak dosyayı Unicode formatında kaydetseydik tekrar açtığımızda düzgün görüntüleyebilecektik çünkü bu karakterin Unicode karakter setinde bir karşılığı bulunmakta. Encoding farkının da aslında verinin nasıl saklandığı konusunda önem arz ettiğini söylemiştim. Bunun örneği için de yine aynı karakteri hem UTF-8, hem de Unicode formatında (UTF-16) kaydettim. UTF-8 formatında 6 byte olarak tutulurken UTF-16 yani Unicode formatında 4 byte olarak tutuluyor. Bunun da non-latin alfabeler ve karakterleri için UTF-16 encoding tipinin kullanılmasının daha uygun olduğunu belirtmiştim.</p>

![img]({{ site.baseurl }}/assets/img/powershell-1/4.png)

![img]({{ site.baseurl }}/assets/img/powershell-1/5.png)

<p align="justify">Bu konular hakkında bilgi sahibi olmak zararlı Powershell Script analizini yaparken ve özellikle bunlar hakkında kendi toolunuzu yazarken önemli ölçüde yardımcı olacaktır. Toollarımı C# diliyle yazdım çünkü Powershell de .NET framework'ü ile hazırlanmış bir script dilidir. </p> 
Olayların arka planda nasıl döndüğünü anlamak için böyle bir tool yazmıştım [Github Repo][gl] . İnceleyerek verinin hex formatında arka planda nasıl tutulduğunu daha rahat anlayabilirsiniz. 

![img]({{ site.baseurl }}/assets/img/powershell-1/6.png)

<p align="justify">Yazdığım her şey çeşitli kaynaklardan edindiğim bilgilerin ve araştırmalarımın sonuçlarının kendimce yorumlanmış halidir. Yanlış veya eksik bir bilgi söz konusuysa lütfen sayfanın en aşağısında bulunan sosyal medya linklerim üzerinden beni bilgilendirin. Bir sonraki yazıda görüşmek üzere. </p>

[cyber-chef]: https://gchq.github.io/CyberChef/
[uni]: https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/
[gl]: https://github.com/batuhankutluca/SystemEncoding/





