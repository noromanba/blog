---
title: "【翻訳】ガーベージコレクションについてよくある誤解"
subtitle: "Translation of \"Everybody thinks about garbage collection the wrong way\""
date: 2015-04-20 12:36:00
---

今週は2010年度CLR週間になりました。今年のCLR週間は例年よりも哲学的な記事を書いていこうと思う。

ガーベージコレクションとは何かと聞かれたとき、大体の人は次のように答えるだろう:「ガーベージコレクションとはプログラムによって使用されなったメモリー領域を実行環境が自動的に回収することである。これはGCルートからメモリーをトレースしてどのオブジェクトが参照されているかを識別することによって実現されてる」

しかし、この説明はガーベージコレクションの目的と実現方法を混同しており、ガーベージコレクションとは何であるかをうまく説明できていない。これでは消防士の仕事を「赤いトラックを運転して水を噴射すること」と説明してるのと同じである。今の説明は消防士がとる行動のいくつかの一例にはなっているが、消防士の仕事の要点を説明できていない。消防士の仕事についてのもっとうまい説明は「消火活動を行うこと」、より広くいえば「火災予防活動を行うこと」である。

ガーベージコレクションのより的確な説明は「無限のメモリーを搭載したコンピューターの擬似的な再現」である。他に付随する説明は、全てその実現方法にすぎない。そしてそれは「消えてもプログラム側に影響をしないようなメモリー領域を回収すること」によって実現されている。このような実現方法はas-ifルール[^1]の応用の大きな一例である。

ガーベージコレクションの真の定義がわかった今、直ちに次のことが導ける:

> もし実行環境で使用可能なRAMがプログラムに必要なメモリーの量を超えている場合、何も行わないガーベージコレクターを搭載したメモリマネージャーもメモリーマネージャとして有効である。

これが本当だと言えるのはメモリーマネージャーは必要に応じてプログラムにRAMを割り当てることができるからであり、仮定からメモリーの割り当ては常に成功する。プログラムのメモリー要件よりも多くのRAMを搭載してるコンピューターは実質無限のRAMを搭載しており、追加で再現する必要はない。

もっとも、今のは当たり前のように思えるかもしれないが、便利な面もある。それは何もしないガーベージコレクターは非常に簡単に解析でき、かつ大体の人が見慣れてるようなガーベージコレクターとは異なるからである。したがって、以下のようなルールが導ける:

> 正しく記述されたプログラムは実行終了前のどのタイミングにおいてもファイナライザーが呼びだされるとは想定できない


このルールが成立するのは、メモリーマネージャーは必要に応じていつでもプログラムにメモリーを割り当てることが可能であるため、メモリーの確保は常に成功するからである。このような状況のもとでは、何もしないガーベージコレクターでも有効であるし、何もしないガーベージコレクターはファイナライザーを走らせないからメモリー領域の回収も一切行われない。

ガーベージコレクションは無限のメモリーを再現するが、無限のメモリーを保有したとしても他のプログラムや場合によっては自分のプログラムにも悪影響を及ぼしてしまうことがある。ファイルを排他的に開いた場合、ファイルを閉じるまでは他のプログラムや自分のプログラムの別の部分でさえもそのファイルにアクセスすることは不可能だ。SQLサーバーとのコネクションを開いた場合、それを閉じるまではサーバー上のリソースを消費し続ける。このようなコネクションをいくつも放置し続ければそれ以上のコネクションが制限されてしまうかもしれない。このようなリソースを明示的に開放しなかった場合、「無限」のメモリーを搭載するようなコンピューターならばいつまでも未開放のリソースが蓄積されて一生開放されることはない。

___ここが開発者が気をつけるべき点___:プログラムを書くとき、ファイナライザーが全てを片付けてくれることを期待してはいけない。ファイナライザーは安全装置であってリソースを開放するための主な手段ではない。用済みのリソースに対しては `Close` や `Disconnect` など、そのオブジェクトを片付けるために使用できるメソッドを明示的に呼び出す必要がある。(`IDisposable` インターフェースはこの取り決めを体系化してる)

更にいえば、正しくプログラムを書こうと思えばプログラムの実行中のみならず、プログラムの実行終了後もファイナライザーが実行されると想定してはならない。.NETフレームワークは極力全てのファイナライザーを実行しようと努力はするものの、ファイナライザーに問題が生じた場合.NETフレームワークはファイナライザーの実行を諦めてしまう。これは開発者の責任でなくとも起こりうることである。例えばファイナライザーがネットワーク上の資源のハンドルを開放しようとしたが、ネットワーク接続に問題が生じたため完了に2秒以上かかってしまう場合、.NETフレームワークはハンドルが開放される前に諦めてプロセスを終了してしまう。したがって先ほどのルールは.NETフレームワークを使用した場合にはさらに厳しくなる:

> 正しく書かれたプログラムはファイナライザーが一切動く保証が無いという仮定のもとで作らないといけない。

以上の知識をもってすれば、以下のように悩んでる顧客の問題を解決することができる。(紛らわしい専門用語はそのまま引用されてる)

> 私は XmlDocument を使用するクラスを扱っています。このクラスがスコープから外れた場合にファイルを削除したいのですが、 "System.IO.Exception: ファイル 'C:\ファイルへのパス.xml` は他のプロセスが使用中であるため、このファイルにアクセスできません。" のような例外が発生してしまいます。プログラムを終了したら、ロックは解除されます。ファイルがロックされてしまうのを防ぐ良い方法は無いでしょうか?

以下の様な解決案では解決する可能性もあるが解決しないかもしれない:

> 私の同僚は用済みになった XmlDocument 変数を null に設定するよう助言しましたが、クラスのスコープから外れた場合と結局同じではないでしょうか?

[^1]:表面上結果が変わらないなら、内部の実装は問わないということ。
元記事: [The Old New Thing: Everyone thinks about garbage collection the wrong way](//blogs.msdn.com/b/oldnewthing/archive/2010/08/09/10047586.aspx)
