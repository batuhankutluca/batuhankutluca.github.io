---
layout: post
title: Ninja
img: "assets/img/portfolio/ninja.png"
date: April, 08 2014
tags: [Lorem]
---

![image]({{ page.img | relative_url }})



It is a python parser to parse hashes in [RedDrip's APT_Digital_Weapon repository.][orig] These hashes are used in APT Attacks and with my parser, you can easily parse all of these hashes and import in your SIEM product for detection. First, you have to fork this repo to your own github account without changing the repository name. Additionally, you need an access token which has "repo" permission scope. Don't forget that if original repository is updated, your forked repository is not updating itself automatically. You need to sync your forked repository with original repository via git.

You can access to my parser via [here.][repo]


[orig]: https://github.com/RedDrip7/APT_Digital_Weapon/tree/master/
[repo]: https://github.com/batuhankutluca/Github-APT-Repo-Hash-Collector
