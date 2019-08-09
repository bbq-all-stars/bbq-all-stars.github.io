---
layout: post
title: phpenvでPHPのバージョンを切り替えたら、brewのicu4uのバージョン違いで怒られた話。
date: 2019-08-09 19:21:00 +0900
---

最近MacでPHPで開発することが多く、anyenv(phpenv)で複数バージョンをインストールして、切り替えて開発することが多いんだけど、この前久しぶりにphpenvで7系から5系に切り替えた時に、以下のエラーが出た。

```
 % php -v
dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicui18n.63.dylib
  Referenced from: /Users/shun/.anyenv/envs/phpenv/versions/5.6.37/bin/php
  Reason: image not found
zsh: abort      php -v
```

ググると割と良くある話なようで、この場合は、php5.6で使うicu4cのバージョンが63系なのだが、64系のバージョンがインストールされていることに起こる問題らしい。  
なので、大体の記事↓はhomebrewでicu4cの古いバージョン（この場合は63系）を入れて、`brew switch`すればいいよ、と書いてある。

[https://stackoverflow.com/a/55828190/11119262](https://stackoverflow.com/a/55828190/11119262){:target="_blank"}

で、このstackoverflowにしたがってやってみたところ、手順2で過去のicu4cのcommit idの確認で詰む。

```
commit c81a048b0ebea0ba976af220806fb8ef35201e9a
Author: BrewTestBot <homebrew-test-bot@lists.sfconservancy.org>
Date:   Fri Apr 19 03:35:49 2019 +0000

    icu4c: update 64.2 bottle.

commit 44895fce117ab92a44d733315b702c48dbb3898d
Author: Chongyu Zhu <i@lembacon.com>
Date:   Thu Mar 28 09:47:20 2019 +0800

    icu4c 64.2

commit 4b5280781960eab6b65ca862707534d5cee4cfc6 (grafted)
Author: BrewTestBot <homebrew-test-bot@lists.sfconservancy.org>
Date:   Fri Apr 12 07:13:00 2019 +0000

    rust: update 1.34.0 bottle.
```

ねえよ！！！！！63系のバージョンがねえよ！！！！！！！！

で、どうしたかというと、実は手順1のFormulaのicu4c.rbファイル、install時に外部から与えることができる↓。

[https://stackoverflow.com/a/56694177/11119262](https://stackoverflow.com/a/56694177/11119262){:target="_blank"}

ということで以下のコマンド。

```
brew install https://raw.githubusercontent.com/Homebrew/homebrew-core/e7f0f10dc63b1dc1061d475f1a61d01b70ef2cb7/Formula/icu4c.rb
```

これで無事に63.1のバージョンのicu4cが入った。

あとはswitchしてやれば...

```
 % brew switch icu4c 63.1
Cleaning /usr/local/Cellar/icu4c/64.2
Cleaning /usr/local/Cellar/icu4c/63.1
Opt link created for /usr/local/Cellar/icu4c/63.1
brew switch icu4c 63.1
 % php -v
PHP 5.6.37 (cli) (built: May 31 2019 17:10:29)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
    with Xdebug v2.5.5, Copyright (c) 2002-2017, by Derick Rethans
```

無事にPHPが動く。
