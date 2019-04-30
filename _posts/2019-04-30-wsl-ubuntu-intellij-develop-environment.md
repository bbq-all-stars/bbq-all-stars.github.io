---
layout: post
title: WSLでIntelliJを動かして、Windowsでも最高の開発環境を手に入れる
date: 2019-04-30 21:00:00 +0900
---

## Windowsがつらい
別にWindowsが嫌だってわけじゃないんだけど、サーバーのアプリケーションはLinux（UNIX）で開発することが多いもんだから、Windows環境で開発しようとするといろいろ問題出るよね。

開発している言語で閉じたアプリケーションならいいんだけど、中途半端にファイル操作とかしていると、パスの指定方法がそれぞれで違ったりしていて地味につらい。

だいたいmacで開発しいている時とかって、あんまりWindowsのこと考えてないよね。

俺考えてないし（考えろ）

## Terminalで開発するという方向
で、じゃあ全部terminalでvim開いて開発すればいいじゃん、ってのも別にできなかないけど、IDEの犬となった今となってはデバッガとか使いたいじゃん？  
それも化石みたいな動的片付け言語でこそデバッガを使いたいわけですよ。

なのにそういう化石みたいな言語ときたら、言語のライブラリの癖に色んなOSのライブラリとかに依存してるわけ。

何なの？死ぬの？って感じ。

となると、WindowsでもIDEが動くUNIX的な環境欲しいじゃん？ってことでタイトル。

これでGW2日も潰したぜ。。。

## 作ったもの
![](/assets/posts/20190430/intellij.png)

これ、IntelliJ IDEAのウィンドウが表示されてるけど、XServerを介してWSL（Windows Subsystem Linux）側で動いているやつ。

だからIntelliJ上でterminalを開くと、WSL上のディレクトリをルートにして開く。  
同じくIntelliJ上でgitを開くと、WSL上のファイルになり、ファイルパーミッション設定とかもWSL上そのまま。

あれ？これ最高じゃない？~~そもそもmacだったら普通なのに~~

## 作り方
筆者の環境はWindows10、WSLはUbuntu 18.04なのであしからず。

参考にしたのはこの辺。
* [Windows Subsystem for Linux + X Windowを1.024倍くらい使いこなすための方法](https://qiita.com/nishemon/items/bb3aca972404f68bfcd6)
* [WSL で Ubuntu デスクトップ環境を作ってみる](https://tmtms.hatenablog.com/entry/201812/wsl-ubuntu)
* [お前らのWSLはそれじゃダメだ](https://xztaityozx.hatenablog.com/entry/2017/12/01/001544)

### Ubuntuにmate（デスクトップ環境）をインストール
別にmateじゃなくてもいいんだけど、とりあえずUbuntuのデスクトップ環境をインストール。

```bash
sudo service dbus start
sudo apt install ubuntu-mate-desktop mate-desktop-environment mate-common mate-core
```

最初にdbusを機動しているのは、こいつを動かしていないとエラーでインストールできなくなるため。

クッソ時間かかる。

実は、もしかしたらXServerとIntelliJだけインストールすればいいのかも知れないけど、GUIでデスクトップ環境があった方が動作確認も含めて何かとよくない？と思ったので、インストール。

mate以外のデスクトップ環境が欲しい人はそっちを入れてくれ。

### WindowsにVcXsrv（XServer）をインストール
Linux側がディスプレイに出力したのを受け取って表示する側のやつ。

[**ここ**](https://sourceforge.net/projects/vcxsrv/)からインストール。

### mateの動作確認
VcXsrvをインストールしたらとりあえず起動。

起動時の設定はとりあえず以下。

* One large Window
* Start no client
* Native opengl以外全部チェック

ちなみにVcXsrvのプログラム名はXLaunchなので気をつけられたし。

起動したら、これとは別にUbuntuのコンソールを開いて以下のコマンド。

```bash
export DISPLAY=:0.0
mate-session
```

するとさっき起動したXLaunchの画面にmateのデスクトップ環境が表示される。

![](/assets/posts/20190430/mate.png)

~~起動すると突然のエラーの洗礼を受ける~~

なんかエラーが出てるけど今のところ問題はない。
多分。

起動が確認できたらXLaunchとmate-sessionをいったん閉じる。

### 日本語環境の整備
せっかくのGUI環境なので、日本語表示の設定と日本語入力の設定を行う。

Ubuntuの日本語パッケージのインストールと設定。
```bash
sudo apt-get install $(check-language-support -l ja)
sudo update-locale LANG=ja_JP.UTF-8
```

日本語入力のパッケージをインストール。  
今回は何も考えずに参考記事にあったuimにした。
```bash
sudo apt install uim-fep uim-anthy
```

uimをインストールしたら、以下のuimの設定をbashrcとかに書く。
```bash
export XIM=uim
export XMODIFIERS=@im=uim
export UIM_CANDWIN_PROG=uim-candwin-gtk
export GTK_IM_MODULE=uim
export QT_IM_MODULE=uim
```

書いたらsource ~/.bashrcしておく。

一通り終わったら同じ手順でmateを起動。

System SettingのLanguage Supportで「日本語」をドラッグして一番上に移動して、日本語設定に。

![](/assets/posts/20190430/mate-jp.png)

あと文字入力設定をuimにする。

設定したらUbuntuの再起動。

コマンドプロンプトで以下のコマンドを打つとUbuntuが再起動する。  
これ以外の再起動方法ないのかなあ。

```
wslconfig /t Ubuntu
```

ちなみにwslconfigの/tオプションは、Windows 10 October 2018 Updateを適用しないと出てこないっぽい。  
アップデートしておくべし。

Ubuntuの再起動が終わったら、同じ手順でmateを起動して確認。  
日本語の表示と入力ができていればOK。

## IntelliJのダウンロード
mateでFirefoxとかを開いて、[ここ](https://www.jetbrains.com/idea/download/#section=linux)からIntelliJをダウンロードする。

ダウンロードしたtar.gzファイルを解凍して適当なところに置く。

解凍したディレクトリの中の`bin/idea.sh`のシェルスクリプトを実行するとIntelliJが起動する。

![](/assets/posts/20190430/mate-intellij.png)

すごい。

実行できることを確認できたら閉じる。  
mateも閉じる。

## 起動用スクリプトを作成
ようやくここからが本番。

Windows側から他のアプリケーションと同等に、アイコンのダブルクリックで一発で開けるようにしたい。

これをやるには以下の2点を利用する。

* XLaunchのconfigを保存しておくと、保存したファイルのダブルクリックで一発で該当の設定で起動できる
* XLaunchのconfigは任意のUNIXコマンドを実行できる

まずは適当な場所にIntelliJ実行用のシェルスクリプトを作る。

```bash:launch_intellij.sh
#!/bin/bash
export XIM=uim
export XMODIFIERS=@im=uim
export UIM_CANDWIN_PROG=uim-candwin-gtk
export GTK_IM_MODULE=uim
export QT_IM_MODULE=uim
export DISPLAY=:0.0
uim-xim &
dbus-launch "/home/shun/Programs/idea-IU-191.6707.61/bin/idea.sh"
```

`uim-xim &`これでuim付きで起動しつつ、dbus-launchコマンドでintellijのシェルスクリプトを指定している。

次にXLaunchの設定ファイルを用意。  
XLaunchを起動して、以下の設定で設定ファイルを保存する。

* Multiple windowsでDisplay numberを0
* Start program
* Start program on this computerで適当なプログラムを指定
* Native opengl以外全部チェック
* 最後にSave configurationして、適当なところに保存。

保存した.xlaunchファイルを適当なテキストエディタで開く。  
テキストエディタで開くと分かるように、xlaunchはただのxmlファイル。

LocalProgramの箇所を書き換えることで任意のスクリプトを実行できるので、以下のように先ほど作成したシェルスクリプトをbashコマンドで実行する用に書き換えて保存。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<XLaunch WindowMode="MultiWindow" ClientMode="StartProgram" LocalClient="True" Display="0" LocalProgram="bash /home/shun/Programs/launch_intellij.sh" RemoteProgram="xterm" RemotePassword="" PrivateKey="" RemoteHost="" RemoteUser="" XDMCPHost="" XDMCPBroadcast="False" XDMCPIndirect="False" Clipboard="True" ClipboardPrimary="True" ExtraParams="" Wgl="False" DisableAC="True" XDMCPTerminate="False"/>
```

なんでこんなxml修正なんてまわりくどいことをやっているのかというと、XLaunchの起動時に指定できるスクリプトの指定に、文字数制限があるため。  
かなしみ。

## 実行
おもむろに書き換えたxlaunchをダブルクリックして実行。

![](/assets/posts/20190430/intellij-single.png)

動いた。  
うれしみ。

## 結論
IntelliJ最高。