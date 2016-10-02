---
title: "【翻訳】PuTTYを安全にダウンロードすることは事実上不可能である"
subtitle: "Translation of \"Downloading Software Safely is Nearly Impossible\""
date: 2015-05-21 15:00:00
---

想像してみて欲しい。あなたは新品のWindowsパソコンを買ってとても満足してる。[出荷時にNSAが細工](http://www.forbes.com/sites/erikkain/2013/12/29/report-nsa-intercepting-laptops-ordered-online-installing-spyware/)してない確信があり、せいぜいマイクロソフトやレノボがプレインストールしたゴミソフトぐらいしかない。あとはLinux端末に接続するためのSSHクライアントさえ入手すれば完璧だ。SSHクライアントのインストール方法は以下の通りである。

(1) ["windows ssh クライアント"](https://www.google.com/search?q=windows+ssh+client&oq=windows+ssh+client)で検索を行う。
 
(2) 一番上の検索候補である[http://www.putty.org/](http://www.putty.org/)を開く。なおホームページにあるのが不正なマルウェアではなくSimon Tathamが開発した正真正銘のPuTTYであることを確認するため、httpsで接続していることを示す鍵のアイコンがあるかどうかを確かめるが残念ながら無い。Tatham氏が暗号技術の開発者とされていることを考えれば心配だ。
 
(3) それどころか[putty.orgはTatham氏が所有するドメインですらない](http://www.whois.com/whois/putty.org)。現在は"denis bider"なる人よって所有されている。きっと他人の商品名のドメインを占拠してリンクをのせるのが趣味なのだろう。
 
(4) ともかくリンクをたどって[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)に移動する。ちゃんとURLにTatham氏の名前が含まれてるし安心するかもしれないが実際は全く安心できない。ホストネーム以外でサイトの所有者を確認する術はないからだ。[greend.org.ukは現在Richard Ketwell氏によって所有されている](http://www.whois.com/whois/greenend.org.uk)。

(5) httpsの鍵アイコンがあるか今一度確認するが見当たらない。繰り返すが暗号技術やセキュリティ関係のソフトウェアは他のソフトウェア同様、信頼できるサービスによって配信するべきはずでは...?

(6) URLの先頭にに手動で"https://"を追加するもサイトはHTTPS非対応で応答しない。ここでこのサイトがPuTTY公式であるか疑い始める。
![]({{ site.baseurl }}/img/2015-05/putty.png "PuTTYはHTTPSでダウンロードすることができない")

(7) それでも心配無用！下にスクロールすればTatham氏が[http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe.RSA](http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe.RSA)など、PuTTYバイナリーのRSAとDSA署名へのリンクを掲載してることがわかる。なお[earth.liはJonathan McDowell氏によって所有されている](http://www.whois.com/whois/earth.li)ことに注意しよう。署名へのリンクをクリックすれば確かにRSA署名っぽいものは出てくるが誰が何に対して署名を行ったものであるのかを知る術がない。もし何者かがサイトを改ざんしたりMITMするなどしてPuTTYのバイナリを不正なものに置き換えた場合署名を置き換えることも容易いはずだ。

(8) [https://the.earth.li/~sgtatham/putty/latest/x86/putty.exe.RSA](https://the.earth.li/~sgtatham/putty/latest/x86/putty.exe.RSA)に接続してHTTPSによる署名のダウンロードを試みるもサーバーに404を返される[^1]。ますます怪しい。

(9) ここで一休みに[Tatham氏が使用してる署名システムがどれほど(必要以上に)複雑であるかについての本人による説明](http://www.chiark.greenend.org.uk/~sgtatham/putty/keys.html)を読むが、配信手段がなぜ匿名でないのかについては説明されず。

(10) ひょっとしたらTatham氏のPGP鍵はMITのPGP鍵サーバーのような公開鍵サーバーに登録されていないかを確認する。[やはり無い](http://pgp.mit.edu/pks/lookup?search=tatham&op=index)。

(11) ついでにMITのPGP鍵サーバーが何ら認証を行っていない[^2]ことが関係しているか少し考えてみる。

(12) たとえTatham氏のPGP鍵を認証された公開鍵サーバーからダウンロードできたとしてもPGPソフトをダウンロードする手間がかかることを思い出す。ここでGnuPGの説明に時間をさくよりも未認証のPuTTYのバイナリーをダウンロードしたほうが早いと諦める。

(13) 一応Tatham氏は[http://www.pc-tools.net/win32/freeware/md5sums/](http://www.pc-tools.net/win32/freeware/md5sums/)にMD5計算ソフトが配布されていることについて触れている。どこの誰が計算したか確認できないためあまり意味は無いけど、MD5ぐらいは確認するかと一瞬考えてみる。だがwww.pc-tools.netもHTTPS非対応であることを確認してそんな時間の無駄は犯すまいと考えを改める[^3]。

(14) putty.exeをダウンロードし、よく考えてから実行する。putty.exeは実行したWindowsユーザーの権限を全て引き受けることにも注意しよう。あなたの書類やメールを読んだり消したり改変したりするのも、あなた秘蔵のお宝コレクションをWikipediaにアップロードするも自在である。

(15) 何も悪さをしないようにお祈りを捧げる。

(16) 仕方なくputty.exeを実行する。インターネットからダウンロードした未認証のプログラムの管理の下、Linuxサーバーのアカウントに接続する。たとえこれが公式のPuTTYバイナリーであったとしても心配は消えない。何にせよPuTTYの開発者はC言語でRSAを実装できる技術の持ち主ではあっても、どういう時にどんな理由でRSAを使えばいいのかを理解してないのである。(ちなみにこの段階であなたが意図した本物のLinuxサーバーに接続できたかどうかは不明である)

(17) ふと、[Web Crypto](//www.w3.org/TR/WebCryptoAPI/)が意外にも結構魅力的に見えてきたことに気づく。[ネイティブコードの信奉者](//rdist.root.org/2010/11/29/final-post-on-javascript-crypto/)が何を言おうがJavascriptは[同一生成元ポリシー](//developer.mozilla.org/ja/docs/Web/JavaScript/Same_origin_policy_for_JavaScript)で動いているし[Chromeのマルチプロセスモデル](//www.chromium.org/developers/design-documents/process-models)によってサンドボックスされていて、あなたのWindowsユーザーアカウントの権限を全て掌握することはできない。

(18) ひたすら絶望する。

元記事: [https://noncombatant.org/2014/03/03/downloading-software-safely-is-nearly-impossible/](https://noncombatant.org/2014/03/03/downloading-software-safely-is-nearly-impossible/)

[^1]: 翻訳者が2015年5月21日にアクセスしたところ、HTTPSで接続することができた
[^2]: 翻訳者が2015年5月21日にアクセスしたところ、HTTPSで接続することができた
[^3]: 実はPowerShellの[Get-FileHash](//technet.microsoft.com/en-us/library/dn520872.aspx)を使えば外部ツールに頼らなくてもハッシュ値を計算できる模様
