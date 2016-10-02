---
title: "mikutter-osx"
subtitle: "OSXでmikutterライフを満喫するためのプラグイン"
date: 2015-10-22 10:55:00
---

パソコンで使うTwitterクライアントといったら[mikutter](http://mikutter.hachune.net)一択なのは有名だが、OSXでmikutterを使用する場合は不便な点がいくつかあった。いくつか例を挙げると:

* X11上で動作するため、OSX標準の日本語入力が使用できない
* Mac用のアプリとしてパッケージされていないため、LaunchpadやSpotlightから起動できない
* インストール手順が煩雑
* アップデート管理が面倒
* 通知機能と音声機能初期状態では利用できない

最近アップデートしてみたらX11不要で動くようになっていたので、残りの問題をまとめて解決すべく[mikutter-osxプラグイン](https://github.com/midchildan/mikutter-osx)を作成した。インストール手順や使用方法は[ここ](https://github.com/midchildan/mikutter-osx/blob/master/README.ja.md)に書いたので、良かったら目を通してください。この記事ではこのプラグインがどのような仕組みで動いてるかについて触れていきたい。

# mikutterをMacアプリ化する
mikutterをMacアプリ化するに方法として、主に3つの方法が考えられた:

1. XCodeを使って作成する
2. Automatorを使ってmikutterを呼び出す
3. mikutterをMacアプリとしてパッケージ化するスクリプトを組む

今回は3番目の方法を採用した。理由としては

* mikutter本体の入手と更新はgitを使って手元で行う方針でいた
* mikutter本体から直接アイコンやバージョン情報を抽出し、Macアプリ用で使用されるicns形式のアイコンとアプリのメタデータを自動的に作成したかった
* mikutter本体はアプリの中にパッケージ化したかった
* mikutterプラグインとして実装したかった (←__重要__!!)

などの理由が挙げられる。

ここでMacアプリがどのような構造になっているか調べるためにAppleの[公式ドキュメント](https://developer.apple.com/library/mac/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html)を参考にしつつ、各種ファイルをどのように配置するか決めた。OSXのアプリは一見単体のファイルのように見えるが実態はディレクトリになっていて、その下に各種ファイルが格納される。このパッケージ方法をAppleはアプリケーションバンドルと呼んでいる。 `mikutter.app` の場合は以下のような構成になっている:

<dl>
  <dt>Contents/</dt>
  <dd>このディレクトリの中に全てのファイルが配置される。古いMacアプリと区別するために必要。</dd>
  <dt>Contents/Info.plist</dt>
  <dd>アプリに関する情報が記述される。</dd>
  <dt>MacOS/</dt>
  <dd>実行可能ファイルが格納される。Unixの<code>bin/</code>と同じような役割をもつ。</dd>
  <dt>MacOS/mikutter</dt>
  <dd>mikutterを起動するためのシェルスクリプト。</dd>
  <dt>Resources/</dt>
  <dd>各種データが配置される。</dd>
  <dt>Resources/mikutter.icns</dt>
  <dd>mikutterのアイコンファイル。</dd>
  <dt>Resources/mikutter/</dt>
  <dd>mikutter本体。</dd>
</dl>

mikutter-osxプラグインに同梱されてる `install.rb` を実行するとgitでmikutterがダウンロードされる。次にmikutter本体にある `core/config.rb` からバージョン情報を読み取って `Info.plist` に記載したり `core/skin/data/icon.png` をApple Icon Imageフォーマットに変換し `mikutter.icns` として保存したりする。`mikutter.app` がなるべく自己完結して実行できるよう必要なgemを流し込んだあと、完成したmikutterバンドルはdmgに詰め込まれデスクトップに保存される。

![]({{ site.baseurl }}/img/2015-10/mikutter_app.png "生成されたmikutter.app")

# アップデートの確認
Linuxディストロのパッケージマネージャーを使ってmikutterをインストールした場合はアップデートが容易であるが、OSXの場合はそうはいかない。OSXでよく使用される[Homebrew](http://brew.sh)のパッケージが従うべき[ガイドライン](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/Acceptable-Formulae.md)を読むと

* 本体とは別に何かをダウンロードするようなスクリプトの使用はダメ
* 安定版がないとダメ
* ニッチなものはダメ

とか書いてあり

* Bundlerを使ってる
* 安定版が[存在しない](http://mikutter.hachune.net/download)
* 日本人以外への知名度が低い

という要素を兼ね備えるmikutterにとって非常に風当たりが強い内容になってる。おまけには

* Appバンドルを作るものはダメ

とも書いてあるので今回作ったようなプラグインまで含めるとHomebrewに受け入れられるようにするのは難しいと思われる。したがってmikutter-osxではmikutterの起動時にアップデートが無いか確認し、アップデートが存在した場合にはインストール時のように自動的にAppバンドルを作ってデスクトップに配置するようにした。アップデートを確認する手段としてはgitを使っていて、mikutter本体のパスを知る必要があるため、mikutter-osxはmikutter本体直下にある `plugin` ディレクトリに配置される。通常のmikutterプラグインと同様に `~/.mikutter/plugin` に配置した場合にはうまく機能しないので注意が必要だ。

# 通知機能と音声再生機能
通知機能に関しては[mikutter-growl](https://github.com/toshia/mikutter-growl)というプラグインが存在するがこれは[growl](http://growl.info)を起動しないといけない。代わりの方法がないか調べてみたら[terminal-notifier](https://github.com/julienXX/terminal-notifier)を使うと良いといった情報がStack Overflowなどでちらほら見られたため、これを採用した。

音声を再生する方法に関しては[ここ](http://dev.mikutter.hachune.net/issues/380)で見つかった。mikutterに同梱されてるALSAプラグイン( `core/plugin/alsa/alsa.rb` )が使用してる `aplay` コマンドの代わりに `afplay` コマンドを使用すれば良いみたいで、これを組み込んだ。

![]({{ site.baseurl }}/img/2015-10/mikutter_notify1.png "mikutterによる通知その1")
![]({{ site.baseurl }}/img/2015-10/mikutter_notify2.png "mikutterによる通知その2")

# 最後に
現段階ではGTK周りの設定を弄らないと見た目が少しみすぼらしいのでGTKのテーマを同梱したい。個人的には今のOSX版Inkscapeに使われてるテーマとかいいんじゃないかと考えてる。

![]({{ site.baseurl }}/img/2015-10/mikutter.png "mikutter on OSX")
