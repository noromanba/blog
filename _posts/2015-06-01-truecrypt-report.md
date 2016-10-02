---
title: "【翻訳】Truecryptに関する報告"
subtitle: "Translation of \"Truecrypt report\""
layout: post
date: 2015-06-01 01:56:00
---
数週間前、私は[Truecryptのオーディットに関する近況](//blog.cryptographyengineering.com/2015/02/another-update-on-truecrypt-audit.html)について書き、じきにはっきりとした結果を見せることができると約束した。NCCクリプト・サービス・グループの努力により遂にその結果を報告できる時が来た。これを可能にしたOCAPのアレックス、シーン、トムとケン・ホワイトに私たちは感謝している。
[完全なレポート](//opencryptoaudit.org/reports/TrueCrypt_Phase_II_NCC_OCAP_final.pdf)は[オープン・クリプト・オーディットプロジェクトのサイト](//opencryptoaudit.org/)で見ることができる。自分で確かめたい人はぜひそうすることを勧める。この記事では簡単な要約に留める。

結論だけ言えば、Truecryptは比較的良く出来た暗号ソフトウェアであるようだ。NCCによるオーディットでは意図的に導入されたバックドアや多くの場合にセキュリティを下げてしまうような深刻な設計上の欠陥を示す証拠は見つからなかった。

ただしそれはTruecryptが完璧であるという訳ではない。オーディットを行った人はいくつかのバグと軽率なコードをいくつか発見した。これにより特定の条件下ではTruecryptによって保証される機密性が望ましい基準を下回る可能性がある。

例を示すと、Truecryptの報告書にあった一番の問題はWindows版Truecryptの乱数生成器に関係している。乱数生成器はTruecyptボリュームを暗号化するためのキーを生成する役割を持つ。予測可能な乱数生成器は他のすべてのセキュリティに影響を及ぼしてしまうため、非常に重要な部分である。

Truecryptの開発者は乱数生成器を[1998年にピーター・ガットマンが考案した仕組み](//www.usenix.org/legacy/publications/library/proceedings/sec98/full_papers/gutmann/gutmann.pdf)を元に実装してる。具体的には *エントロピー・プール* を使って[Windows Crypto API](//msdn.microsoft.com/en-us/library/ms867086.aspx)を含めたシステムのあらゆるソースから予測不能な値を集めている。Truecryptで問題なのは、ごく稀にCypto APIがきちんと初期化するのに失敗することがあるということだ。このようなことが起きれば、本来Truecryptは検知して対処しなければならない。しかし、Truecryptは構わずキーを生成し続けてしまう。

```c
/* CryptoAcquireContextの呼び出し */
if (!CryptAcquireContext (&hCryptProv, NULL, NULL, PROV_RSA_FULL, 0)
	&& !CryptAcquireContext (&hCryptProv, NULL, NULL, PROV_RSA_FULL, CRYPT_NEWKEYSET))
	CryptoAPIAvailabe = FALSE;
else
	CryptoAPIAvailable = TRUE;
```

ただし、このような失敗の可能性は非常に低いため、世界の終わりという訳では無い。また、Windows Crypto APIが仮にあなたのシステムで失敗を起こしたとしてもTruecryptはシステムポインターやマウスの動きなど、他のソースからエントロピーを獲得する。このような代替手段はあなたを守るのにおそらく十分であるが設計上問題があり、Truecryptのフォークでは確実に直すべきだ。

乱数生成器に関する問題の他にも、TruecryptのAESコードが *キャッシュタイミング攻撃* に反応しやすいという懸念についても触れられていた。これは共用のコンピューターや攻撃者がコードを実行できる環境(例えばサンドボックス内や可能性としてはブラウザー内)で暗号化・復号化を行わない限りはおそらく心配ない。だが、これはTruecryptを元にして作られるプロジェクトの指針にはなるはずだ。

Truecryptは実に特別なソフトウェアであった。Truecryptの開発者の損失はデータの保護にフルディスク暗号化を頼る多くの人々に痛烈に実感されている。運が良ければTruecryptのコードは他の人に引き継がれるかもしれない。この調査がその手助けになれば何よりである。

元記事: [Truecrypt report - A Few Thoughts on Cryptographic Engineering](//blog.cryptographyengineering.com/2015/04/truecrypt-report.html)
