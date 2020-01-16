---
layout: post
title: "Tweepy ile Python Twitter Modülü"
feature-img: 'assets/img/tweepy.png'
date: 2018-04-12 01:14:00
author: Batuhan Kutluca
tags: [Tweepy, Python, Twitter]
---

<p align="justify">Herkese selam, ilk yazıma hoşgeldiniz. Tweet atabilen, tüm tweetlerimizi ve favorilerimizi silebilen ve bir kullanıcıyı takip edebilen ve takipten
çıkarabilen bir Python scripti yazacağız. Bu scriptimizi yazarken Twitter API'ını kullanan "Tweepy" adlı kütüphaneyi kullanacağız.</p>

| Kütüphane | Link |
| ------ | ------ |
| Tweepy | [https://github.com/tweepy/tweepy](https://github.com/tweepy/tweepy) |

<p align="justify">Öncelikle tweepy kütüphanesini kullanabilmek için gerekli kurulumu yapmamız lazım. Konsol ekranına</p> 

```python
pip3 install tweepy
```

<p align="justify">yazarak kütüphanenin kurulumunu yapabiliriz. Daha sonra kütüphanemizi ekleyip gerekli ayarlamaları yapmamız gerekiyor.</p>

```python
import sys, tweepy, time
consumerKey = "Your consumerKey here!"
consumerSecret = "Your consumerSecret here!"
accessToken = "Your accessToken here!"
accessTokenSecret = "Your accessTokenSecret here!"
```

<p align="justify">Buradaki 4 Token kullanıcıya özel tokenlardır. Bu tokenları 'https://apps.twitter.com/' adresinden yeni bir uygulama oluşturup alabiliriz.
Uygulamamızı oluşturduktan sonra "Keys and Access Tokens" sekmesinden keylerimizi alıyoruz ve kodumuzda gerekli yerlere yazıyoruz. Bu scripti yazmamın asıl amacı eski Twitter hesabımdaki tweetlerimi silmekti. Ancak Tweepy kütüphanesinin dökümantasyonuna biraz göz attığımda birkaç özellik daha eklemeye karar verdim. Gerekli kütüphaneleri ekledikten sonra kütüphanenin özelliklerini kullanabilmemiz için auth olmamız lazım. Bir auth fonksiyonu yazalım ve dönen değeri bir değişkende tutalım. Çünkü dönen değeri kullanarak API'ımızın bize sunduğu fonksiyonları kullanabiliriz.</p>

```python
def login(consumerKey, consumerSecret, accessToken, accessTokenSecret):
    auth = tweepy.OAuthHandler(consumerKey,consumerSecret)
    auth.set_access_token(accessToken,accessTokenSecret)
    api = tweepy.API(auth)
    return api
```
<p align="justify">Önce başarılı login olup olmadığını kontrol edip daha sonra modülümüzü tasarlayabilirdik ancak modül zaten kendimize ait olacağı için bunu çok gerekli görmedim. Bu yüzden modülümüzün her bir fonksiyonunda önce login fonksiyonundan dönen değeri bir değişkende tutup daha sonra bu değişken doğrultusunda API'ın özelliklerini kullanacağız. login fonksiyonunu yazdıktan sonra main fonksiyonumuzu yazalım ve scriptimiz çalıştığında kullanıcıya hangi seçenekleri sunacağına karar verelim.</p>

```python
def main():
    print("Type 'tweet' to send tweet.")
    print("Type 'delete_tweets' to delete your tweets.")
    print("Type 'follow' to follow someone.")
    print("Type 'unfollow' to unfollow someone.")
    print("Type 'delete_favs' to delete your favourites.")
    print("Type 'exit' to exit from script.")

    choice = input()
    if (choice.lower() == "tweet"):
        tweet(input("Please type your tweet : "))
    elif (choice.lower() == "delete_tweets"):
        delete_tweets()
    elif (choice.lower() == "follow"):
        follow(input("Please type username :"))
    elif (choice.lower() == "unfollow"):
        unfollow(input("Please type username :"))
    elif (choice.lower() == "delete_favs"):
        delete_favs()
    elif (choice.lower() == "exit"):
        sys.exit
    else:
        print("Please make your choice correctly.")

if __name__ == '__main__':
    main()
```
<p align="justify">Kullanıcıdan bir seçim yapıp yapılan seçim doğrultusunda hangi fonksiyonları çağıracağımıza karar verdik. Bütün fonksiyonları buraya yazıp tek tek açıklamaktansa örnek olması açısından delete_tweets fonksiyonunu hep beraber inceleyelim.</p>

```python
def delete_tweets():
    apii = login(consumerKey, consumerSecret, accessToken, accessTokenSecret)
    timeline = tweepy.Cursor(apii.user_timeline).items()
    for status in timeline:
        apii.destroy_status(status.id)
        time.sleep(1)
    print("All tweets have been deleted. \n")
    main()
```
<p align="justify">Üstte açıkladığım gibi login fonksiyonundan dönen değeri 'apii' adlı bir değişkene atadık. Daha sonra tweetlerimizi timeline adında bir listede tuttuk. Listemizdeki her bir item yani tweet için tweetimize özel status.id değerini 'destroy_status()' fonksiyonuna parametre olarak göndererek tweetlerimizi sildik. Bir anda çok fazla istek gitmemesi için bu işlemi 1 sn arayla yaptık ve bunun için 'time.sleep(1)' kodunu ekledik. Daha sonra tekrar scriptimizin seçim sayfasına döndük.</p>

Scriptin tamamına [linke](https://github.com/batuhankutluca/Python-Twitter-module) tıklayarak ulaşabilirsiniz. Bir sonraki yazıda buluşmak dileğiyle hoşçakalın.


