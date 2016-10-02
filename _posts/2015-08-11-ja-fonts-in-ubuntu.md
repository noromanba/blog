---
title: "ロケールとフォント"
subtitle: "英語ロケールで日本語を綺麗に表示する方法について"
date: 2015-08-11 23:40:00
---
日本だと自分の使用する端末のロケールは日本語か英語のどちらかにしてる場合が多いだろう。私の場合は英語にする場合が多い。なぜならば普段使用するソフトウェアの多くが英語圏を中心に開発され日本語に翻訳されたものである。それでもただUIが翻訳されただけならたまに少し変な日本語が表示されるだけで済むが、実際はそうではなく作成されるファイルの名前まで日本語化される場合が多々あってうんざりする。そしてこれは主観的な問題であるが、ロケールが英語だと特にメニュー周りがすっきりして見やすく見栄えも良い場合が多い。ただしロケールを英語にした場合に気になるのが日本語フォントだ。厄介なことにCJK文字は重複する文字が多くフォントの設定を統一することが非常に困難であり、ロケールに合わせて設定されている場合が多い。したがって英語ロケールの下では日本語の見栄えが悪くなることがある。

残念ながらUbuntuもロケールを英語にすると日本語フォントが悪くなる。そして今回対処方法を調べたが、こんなことで悩むのはCJKユーザーぐらいであり情報が少なく苦戦した。他にも似たような問題で悩む人は多いと思うので今回調べた結果をここに記しておく。

# 従来の方法
Linux環境ではフォントの設定はFontconfigで管理するのが主流になっている。フォントの設定がどのようにされてるかについては[この記事](//gihyo.jp/admin/serial/01/ubuntu-recipe/0039?page=2)などに任せる。古いバージョンのUbuntuだと、日本語フォントに関する設定は `/etc/fonts/conf.avail/69-language-selector-ja-jp.conf` などのファイルにまとめられ、ロケールが日本語に設定されたら `/etc/fonts/conf.d` にシムリンクを張ることで有効にされていた。したがって英語ロケールでフォント設定を日本語向けにしようと思った場合、手動でリンクを張るなどしてこの設定を有効化するなどの方法が取られていた。

しかし今では[状況が変化](//gihyo.jp/admin/clip/01/ubuntu-topics/201208/24)し、 `/etc/fonts/conf.avail/69-language-selector-ja-jp.conf` などのファイルは廃止されてる。そこで古いバージョンのUbuntuからこのファイルを引っ張りだして適用する方法を取る人がいるが、オススメ出来ない。すでに廃止されたものを強引に適用するこの方法では根本的解決にはならないからだ。

# 今回の解決策
では従来まとめれていた日本語向けフォント設定は一体どうなってしまったのかというと、各フォント個別の設定として分散されている。例えば、Takao ゴシックというフォントに関する設定が記述されてる `/etc/fonts/conf.avail/65-fonts-takao-gothic.conf` を見ると次のような記述がある:
```xml
<match target="pattern">
  <test name="lang" compare="contains">
    <string>ja</string>
  </test>
  <test qual="any" name="family">
    <string>monospace</string>
  </test>
  <edit name="family" mode="prepend" binding="strong">
    <string>TakaoGothic</string>
  </edit>
</match>
```
この設定により日本語環境下ではmonospaceフォントととしてTakao ゴシックが優先されるようになる。すなわち、ロケールに合わせてフォント選択の挙動が変化するように設定されてるのである。このようなロケールに合わせたフォント設定に関しては[Fedoraのウィキ](//fedoraproject.org/wiki/Fontconfig_packaging_tips#Locale-specific_overrides)に詳しく書いてあるので参考にするといいだろう。

ではロケールに合わせてフォント選択が変わってしまうならば、どうやって英語ロケールで日本語に適したフォントを選択させればいいのか。それはFontconfigが言語を調べるときの挙動に着目すればいい。[ここ](//www.freedesktop.org/software/fontconfig/fontconfig-devel/fcgetdefaultlangs.html)を見ると、どうやらFontconfigは `FC_LANG` 、 `LC_ALL` 、 `LC_CTYPE` 、 `LANG` の順番に環境変数を調べるようである。したがって、環境変数  `FC_LANG` を日本語に設定していれば `LANG` 英語になっていようと日本語に適したフォントが選択されることになる。試しに異なる環境変数で `fc-match` コマンドを叩き、システムのデフォルトフォントがどう変化するか調べてみた。まずは通常の日本語ロケールの場合から:
```console
$ env -i LANG=ja_JP.UTF-8 fc-match
fonts-japanese-gothic.ttf: "Takao Pゴシック" "Regular"
```
`env` コマンドに `-i` オプションを渡すと空の環境から始めることが出来る。すなわち、 `FC_LANG` が設定されてない状態で `LANG` が `ja_JP.UTF-8` に設定され場合、Takao Pゴシックがデフォルトフォントに選択されることを示してる。次も同様に通常の英語ロケールについて:
```console
$ env -i LANG=en_US.UTF-8 fc-match
DejaVuSans.ttff: "DejaVu Sans" "Book"
```
今回は変わってDejaVu Sansが採用されてるのが分かる。では、英語ロケールで `FC_LANG` を `ja` に設定したらどうだろうか。
```console
$ env LANG=en_US.UTF-8 FC_LANG=ja fc-match
fonts-japanese-gothic.ttf: "Takao Pゴシック" "Regular"
```
ちゃんとTakao Pゴシックが選択され、日本語に適したフォントが選択されてることが分かる。

# 結論
環境変数 `FC_LANG` を `ja` に設定しよう。 `~/.profile` に一行
```sh
export FC_LANG=ja
```
と追加すれば良い。最後に、環境変数を設定する前後を比較してみる。次が設定する前のフォントである:

![]({{ site.baseurl }}/img/2015-08/badfont.png "設定前")

やはりところどころ字がおかしい。そして次が設定後のフォントである:

![]({{ site.baseurl }}/img/2015-08/goodfont.png "設定後")

ちゃんと違和感のないフォントになっていることが分かる。
