---
title: "brew install mikutter"
subtitle: "MikutterInstallBattle on macOS 2016"
date: 2016-11-07 23:35:00
header-img: "img/2016-11/resona.png"
---

今日Homebrewに[mikutterのフォーミュラを追加した](//github.com/Homebrew/homebrew-core/pull/6258)。[mikutter](//mikutter.hachune.net)とはプラグイン拡張可能なツイッタークライアントであり、主に日本のLinuxユーザーに愛用されてる。Debianなどでは[mikutterのパッケージ](//packages.debian.org/ja/jessie/mikutter)が用意されていてコマンド一発でインストールすることができるが、macOSでのインストールは少し面倒な手順を踏む必要があった。そこで以前[インストールスクリプト]({% post_url 2015-10-22-mikutter-osx-plugin %})を書くなどしたがやはり面倒だからmacOSで人気のあるパッケージマネージャーの[Homebrew](//brew.sh)向けにフォーミュラを作ることにした。今回はmikutterのインストール方法とフォーミュラ作成の流れについて書く。

# インストール方法

Homebrewで一発(完)。Homebrewでは[利用が少ないないフォーミュラは無慈悲に消される](//github.com/Homebrew/homebrew-core/issues/6408)のでじゃんじゃんインストールしよう。

```console
$ brew install mikutter
```

あとはターミナルに `mikutter` と打ち込むだけで実行できる。さらに以下の手順を踏めばより使いやすくすることができる。

## mikutter-mac プラグイン

初期状態ではいくつかの機能が使えないため、[mac向けのプラグインを用意した](//github.com/midchildan/mikutter-mac)。これをインストールすると

- 音声の有効化
- 通知の有効化
- Dockにアイコンの表示
- LaunchpadやSpotlightから起動

などが可能になる。

![]({{ site.baseurl }}/img/2016-11/notifications.png "notifications")

## GTK テーマ

GTKテーマを導入することによって見た目を良くすることができる。まずは必要なパッケージを導入する。

```sh
brew install gtk-engines gtk-murrine-engine
```

そして好きなテーマを見つけて導入するれば良い。自分は[Zukitre](//github.com/lassekongo83/zuki-themes)というテーマを利用してる。

```sh
mkdir ~/.themes
cd ~/.themes
git clone https://github.com/lassekongo83/zuki-themes.git
echo "include \"$HOME/.themes/zuki-themes/Zukitre/gtk-2.0/gtkrc\"" > ~/.gtkrc-2.0
```

![]({{ site.baseurl }}/img/2016-11/mikutter.png "mikutter on macOS")

# フォーミュラ作成の流れ

## ドキュメントを読む

最初に[公式ドキュメント](//github.com/Homebrew/brew/tree/master/docs)に目を通した。以下の３つが特に重要だった。

- [Acceptable Formula](//github.com/Homebrew/brew/blob/master/docs/Acceptable-Formulae.md)
- [Formula Cookbook](//github.com/Homebrew/brew/blob/master/docs/Formula-Cookbook.md)
- [How To open a Homebrew Pull Request (and get it merged)](
https://github.com/Homebrew/brew/blob/master/docs/How-To-Open-a-Homebrew-Pull-Request-%28and-get-it-merged%29.md)

細かい書き方などはドキュメントに書かれてなく、レポジトリのコードを参考にする必要があった。しかし最近[Homebrew本体](//github.com/Homebrew/brew)と[そのフォーミュラ](//github.com/Hombrew/homebrew-core)が別々のレポジトリで管理されるようになったため、フォーミュラで使えるメソッドの定義と使用例を別々に検索することができて便利だった。

## 問題点

ドキュメントに目を通したところ、ある問題が浮上した。Homebrewでは `gem` や `pip` などを使って[Homebrewを介さずにダウンロードを行うフォーミュラは基本的に受け付けない](//github.com/Homebrew/brew/blob/master/docs/Acceptable-Formulae.md#we-dont-like-install-scripts-that-download-things)。したがってgemやPythonのパッケージを依存関係に利用する場合はHomebrewの`resource`文を利用して[必要なパッケージごとにURLとハッシュ値を指定する決まりになっている](//github.com/Homebrew/brew/blob/master/docs/Formula-Cookbook.md#specifying-gems-python-modules-go-projects-etc-as-dependencies)。

しかしパッケージの依存関係やそれら全てのURLとハッシュ値を手動で調べるのは大変な手間がかかる。mikutterの場合は最終的に必要なgemが４０個存在し、これを手動でやるとそれなりに時間がかかる。PythonやGoで書かれたものであれば [homebrew-pypi-poet](//github.com/tdsmith/homebrew-pypi-poet) や [gdm](//github.com/sparrc/gdm#homebrew) などのツールが面倒を見てくれるが、Rubyのgemに関してはこういったツールは存在しなかった。そういう訳で `Gemfile`からHomebrewの `resource` 文を自動生成するツール[resona](//github.com/midchildan/resona)を作った。なおresonaという名称は"Homebrew <strong class="text-primary">reso</strong>urce sta<strong class="text-primary">n</strong>z<strong class="text-primary">a</strong> generator for RubyGems dependencies"の略であり決して[大蔵りそな](//project-navel.com/otomeriron/chara01_resona.html)と関係はない。拙者はオタクではござらんので(ｺﾎﾟｫ

また、resonaを元にして[mikutterフォーミュラを更新するスクリプトも公開した](//github.com/midchildan/mikutterbrew)。

## テストの作成

Homebrewのフォーミュラには[パッケージングに問題がないかを確認するためのテストが書かれる](//github.com/Homebrew/brew/blob/master/docs/Formula-Cookbook.md#add-a-test-to-the-formula)。GUIのツイッタークライアントをどうやってテストするか少し悩んだが、GUIなどを無効にしつつ読み込んでからしばらくして終了するプラグインを用意することによって対処した。

```ruby
test do
  test_plugin = <<-EOS.undent
    # -*- coding: utf-8 -*-
    Plugin.create(:brew) do
      Delayer.new { Thread.exit }
    end
  EOS
  (testpath/"plugin/brew.rb").write(test_plugin)
  system bin/"mikutter", "--confroot=#{testpath}", "--plugin=brew", "--debug"
end
```

## プルリクを送る

いくつかHombrewのバグ([brew#1209](//github.com/Homebrew/brew/issues/1209),
[hombrew-core#5413](//github.com/Homebrew/homebrew-core/issues/5413),
[homebrew-core#5482](//github.com/Homebrew/homebrew-core/issues/5482),
[brew#1404](//github.com/Homebrew/brew/pull/1404))を踏むなどしたが、無事マージ
できてよかった。
