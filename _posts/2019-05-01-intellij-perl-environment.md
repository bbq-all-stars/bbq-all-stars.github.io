---
layout: post
title: どうしても業務命令でPerlを使わざるを得ないあなたに送る、IDE（IntelliJ）でPerlを殴る方法。
date: 2019-05-01 16:04:00 +0900
---

今更なんで俺perlの環境をいれてるんだろう。。。  
しかし昔の俺とは違う。  
今はIntelliJがそばについているんだ。  
俺はこいつと一緒にPerlを殴りに出かけた。  

（注）この記事は前職で社内向けに書いたナレッジの修正版です。

# plenv
perlのバージョン管理。  
だいたいこいつのインストールで数分の時間を取られる。  
goのインストール時間をもっと見習って欲しいところですね。  

[https://papix.hatenablog.com/entry/2018/03/10/160838](https://papix.hatenablog.com/entry/2018/03/10/160838)

anyenv入れておくと便利だよ。  

欲しいperlバージョンをインストールしたら、
```perl
plenv install-cpanm
```
してcpanmをインストールしておくこと。

# carton
モジュール管理。  
cpanmとか使ってる人は今すぐ窓から飛んだほうがいい。  
cartonは厳密なバージョン管理で冪等性が保たれるので、必須。  
しかもインストールが並列で行われるので激早い。  

しかしあれだ。carton入れるためにcpanmを入れる必要があるという以下ループ。

インストールは以下のコマンド。
```
cpanm Carton 
```

perlはnodeのpackage.jsonみたいに、cpanfileというテキストファイルでインストールするモジュールを指定する。  
なのでモジュールを追加したかったら、cpanfileに欲しいモジュールを書いて
```
carton install
```
とやると、プロジェクト配下のlocalディレクトリ以下にインストールされる。  
同時にcpanfile.snapshotというファイルも作られ、そこに今インストールされたモジュールのバージョンが書き込まれる。

デプロイ時は、
```
carton install --deployment
```
と打ってやれば、cpanfile.snapshotだけを見てインストールしてくれる。

また、本来cpanmでインストールできる実行コマンドは、cartonの場合、
```
carton exec -- hoge
```

で実行できる。  
例えばplackupなどは
```
carton exec -- plackup
```

みたいな。

他の使い方はググって調べてくれ。

# perltidy、perlcriticのインストール
perltidyはコードフォーマッター。  
perlcriticは静的解析ツール。  
後述するIntelliJで設定しておくと何かと便利。

以前はcartonでインストールする方法書いていたが、IntelliJのアップデートに伴って使えなくなったので、大人しくグローバルにインストールしよう。

```
cpanm Perl::Tidy
cpanm Perl::Critic
```

# Devel::Camelcadedbのインストール
IntelliJでperlをデバッグするならコレ。  
自分のIntelliJのバージョンと同じバージョンをインストールすること。

```
cpanm Devel::Camelcadedb@v2018.3.0
```

↑はバージョン2018.3.0の場合の例。

# IntelliJの設定
IntelliJは神ソフトなので、perlのプラグインで色々できる。  

まずはperlのプラグインをインストール。  

![](/assets/posts/20190501/perl-plugin.png)

次にLanguages & Frameworks -> Perl5のProject設定から、以下を設定。

* Perl5 Interpreter
    * perlのバイナリーが入っているディレクトリを右側のギヤマークから選択する。anyenvでインストールした場合は、Add plenv Perl...を選べばplenvでインストールしたperlが選択できる。
* External libraries
    * 自作ライブラリを使っている場合等はここで設定しておく
* Enable Perl::Critic annotations にチェック

![](/assets/posts/20190501/project-perl-setting.png)

ちなみにperltidyはデフォルトで80文字以上の行は自動改行するようになっているが、80文字だと結構短いので、-lオプションでintellijと同じ設定の120文字に設定している。  
また、perltidyはデフォルトではファイルの上書きをせずにフォーマット結果を標準出力に出すが、-bオプションをつければ上書きの上、上書き前のものを"ファイル名.bak"ファイルとして吐き出すようになる。

次にプロジェクトディレクトリの設定をする。

ライブラリやテンプレートの入っているディレクトリを設定する。  
設定したいディレクトリを選択して、Mark as:から設定したいタイプを選んであげれば良い。  

![](/assets/posts/20190501/project-perl-directory-setting.png)

Perl5 librariesに指定するのは、ローカルのlibディレクトリや、cartonで入ったlocal/lib/perl5ディレクトリで十分だろう。  
macの場合はlocal/lib/perl5/darwin-2level、linuxの場合はlocal/lib/perl5/x86_64-linuxにも入るので、こちらも指定を忘れないように。

また、git submoduleで管理している依存プロジェクトある場合などは、そのプロジェクトのlibディレクトリと、そのプロジェクトのlocal/lib/perl5ディレクトリを設定する。

# 自動でコードフォーマット
↑でperltidyで設定したから保存時に自動でコードフォーマットできるかな、と思ったけど、実はできない。  
上記の設定は、あくまでcmd + option + Lした時にperltidyでフォーマットできるというもの。

自動でやりたい場合、goとかと同じようにFile Watchersに登録する必要がある。  

![](/assets/posts/20190501/filewatcher.png)

これで保存時に自動でコードフォーマットしてくれるようになる。

# .gitignore
以上の設定を行うと、プロジェクトディレクトリ直下の.ideaディレクトリ内に設定ファイルが吐かれる。  
この設定ファイルを共有すると、プロジェクトメンバーに設定ファイルを共有できるのでやるべし。  
間違っても雑に.ideaディレクトリを.gitignoreに追加しないこと。

.ideaディレクトリは無視した方が良いファイルもあるので、[gibo](https://qiita.com/tmknom/items/c4bcebe17d25381fa45d)で.gitignoreを設定する。
```
gibo JetBrains >> .gitignore
```

ちなみに、.ideaのファイルのうち、.idea/watcherTasks.xmlを共有すると誰のPCでも自動でコードフォーマットすることができるようになる。  
ただ、perltidyのパスが他人によって違うと動かなくなるので、うまくいかないならignoreするのが無難だろう。  
（なお.idea/watcherTasks.xmlは、ホームディレクトリからの相対パスで記載されているので、ホームディレクトリからのパスが同じであれば共有して使うことは可能である。）

# debugger
plackupコマンドはperlスクリプトなので、それを実行するようにRun Configurationを設定すれば、psgiファイルをデバッグすることができる。  
psgiファイルを指定してサーバーをデバッグする設定は以下。

![](/assets/posts/20190501/plackup.png)

plackupのスクリプトを指定して、script parametersに実行したいpsgiファイルを指定する。  
あとはEnvironmentでPLACK_ENVを指定したり、LM_DEBUGを指定したりしてよしなに設定を書く。

右側のShareにチェックをつけてgitでファイルを共有するようにすると、全員がすぐにサーバーを起動できるようになるので、しておくべし。

デバッグのやり方は、適当なところにbreakポイントを打って、デバッガーで起動して、あとはアクセスすればbreakポイントで処理が一時停止する。  
これで実行中のhashの中身が見放題だぜ！！！！！！！

![](/assets/posts/20190501/debug.png)

