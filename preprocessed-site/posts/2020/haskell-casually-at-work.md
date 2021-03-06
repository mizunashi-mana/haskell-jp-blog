---
title: Haskellを業務で使う、カジュアルに
headingBackgroundImage: ../../img/background.png
headingDivClass: post-heading
author: takenobu.hs
postedBy: takenobu.hs
date: April 26, 2020
tags:
...
---

はじめに
=======

この記事では、Haskellを業務でカジュアルに使う観点やヒントについて、簡単に紹介します。

Haskellを業務で使える局面は、以下のようにいくつか考えられます。

1. 「プロダクト」の開発用言語として、Haskellを使う
2. 「作業」の支援・加速用に、Haskellを使う
3. 「思考」の支援・加速用に、Haskellを使う

つまり、プロダクトの開発用言語としてHaskellを用いない業務形態においても、上記2や3のケースとして、Haskellを使用できます。すなわち、Haskellは幅広い局面でカジュアルに、つまり気軽に手軽に使用できます。

本記事では、特に、上記の2と3について、いくつかの観点やヒントや例を紹介します。

なお、上記は、Haskellを用いる場合には限りません。Python, Perl, Ruby, Rust, Scala, OCaml, Clojure, Go, Elixir, ... といった、様々なプログラミング言語に置き換えて本記事を解釈してもらって構いません。


------

🔧「作業」の支援・加速に、Haskellを使う
=====================================

Haskell（を含むプログラミング言語）は、開発などの日常業務において、「作業」の支援・加速用に使うことが出来ます。

つまり、電卓やExcelなどのように、Haskellを日常ツールの一つとして使えます。

特に、直近の業務作業を加速するために、書き捨てのツールを高品質で素早く欲しい場合や、ちょっとした対話ツールを欲しい場合などにも、Haskellを便利に活用できます。

例えば具体的には、以下の場合にHaskellを便利に使えます。

  * テストデータ生成
  * パーサー
  * 階層データ処理
  * 高機能電卓

以下、それぞれについて簡単に紹介します。


## テストデータ生成

例えば、解析事案が発生し、至急10分程度でテストデータを複数用意したい、というような場合に、Haskellでデータを生成させることは有効です。

Haskellは、関数合成や部分適用や高階関数や多相関数などの言語的な特徴により、小さな関数を組み合わせて、より大きな関数として作り上げることが容易です。

対話環境（REPL）であるGHCiを用いて、それら小さな関数を素早く高品質に確認した上で、徐々に大きな関数として組み合わせることにより、高品質な結果を素早く得ることがでできます。  

特にバイナリデータや複雑なデータを、一刻も早く高品質に生成することが重要な局面で、Haskellは威力を発揮します。


## パーサー

日常業務において、各種ログなどのデータを解析したい局面は頻繁に有ります。
単純なデータであれば、grepコマンドやPerlなどの正規表現を用いて手早く仕事を済ませることも出来ます。

しかし、データの構造が複雑であったり再帰的な構造である場合には、正規表現をデバッグするよりも、Haskellで思い切ってパーサーを書いてしまう方が手早く済ませられることがあります。

Haskellでは、関数の組み立てが容易であることやdo記法といった言語的な特徴を活かし、簡潔にパーサーを記述することができます。
言語的な特徴を活かした便利なパーサーコンビネータ関連のライブラリ（[`Parsec`](https://hackage.haskell.org/package/parsec)や[`Megaparsec`](https://hackage.haskell.org/package/megaparsec)や[`replace-attoparsec`](https://hackage.haskell.org/package/replace-attoparsec)など）が豊富に存在します。

一度パーサーの骨格を用意してしまえば、流用は容易であるため、強力な日常ツールとしてHaskellを便利に使用できます。


## 階層データ処理

例えばモジュールの構造に対応したデータのように、データが再帰的・階層的に表現されている場合は多くあります。

Haskellは、代数的データ型を用いて再帰的なデータ構造を簡潔に表現できます。また、簡潔なパターンマッチの記法と再帰的な関数により、これらの処理を容易に記述できる傾向にあります。

もちろん、この再帰的なデータ構造も、コンパイル時の静的な型チェックの対象となるため、多くの不用意なミスを事前に抽出できます。

素早く、非常に高品質にデータ処理を行うことが重要な局面で、Haskellは有効に機能します。


## 高機能電卓

日常業務において、なんらかの変換テーブルや、計算式、定数値などの値を、散発的に直ちに得たい局面があります。
その都度、電卓で計算したり、Excelなどの計算フォームを用意することで、手軽に業務を済ませられる場合もあります。

しかし、繰り返し必要となる計算式や、ある程度複雑な計算であれば、これらの計算式などを、Haskellの関数群として定義しておき、対話環境GHCiから用いることで、使い勝手良く素早く値を得ることができます。

数値や対話操作などを補助する便利なライブラリ([`Numeric`](https://hackage.haskell.org/package/base/docs/Numeric.html)や[`Data.Bits`](https://hackage.haskell.org/package/base/docs/Data-Bits.html)や[`Data.GHex`](http://hackage.haskell.org/package/ghci-hexcalc/docs/Data-GHex.html))や言語拡張([`BinaryLiterals`](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/glasgow_exts.html#binary-integer-literals)や[`NumericUnderscores`](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/glasgow_exts.html#numeric-underscores))などが豊富に存在します。

Haskellにおける関数の組み立てが容易な特徴は、対話環境における対話的な操作との相性が良いため、試行錯誤的な計算作業にも有用です。


## その他

他にも、定型的なファイル処理やCLIコマンドやDSLの構築などを、Haskellを用いて便利に実現出来ます。
手元に各種雛形を蓄積していると、作業の素早さと正確さが求められる場合に、有益でしょう。

もちろん、これらはHaskellに限らず、多くのプログラミング言語にも言えます。

Haskellは、型システムに守られながら、関数を容易に組み立てられる特徴を持ちます。また、代数的データ型とパターンマッチの特徴により、直感的・シンプルで高品質なデータ表現・処理が可能です。さらに、GHCiを用いる対話操作により、日常作業を高品質かつ手早く行えます。

Haskellは、（型システムの高度な機能などを使わない）基本的な機能のみにおいても、日常業務において有効に活用できるツールの一つです。


------

💡「思考」の支援・加速に、Haskellを使う
=====================================

Haskell（を含むプログラミング言語）は、開発などの日常業務において、「思考」の支援・加速用にも使うことが出来ます。

つまり、紙と鉛筆などのように、Haskellを思考ツールの一つとして使えます。

特に、試行錯誤的な思考フェーズや、探索フェーズにおいて、思考を整理・加速する場合などに、Haskellは便利です。

例えば具体的には、以下の場合にHaskellを便利に使えます。

  * 仕様理解
  * モデル確認
  * モデル探索

以下、それぞれについて簡単に紹介します。


## 仕様理解

ハードウェアやソフトウェア開発過程などでは、例えば、自然言語と図表や式の組み合わせで表現された仕様書を理解する事が必要な局面が多くあります。

設計の上流工程で思考を広く深く及ばせておくことにより、仕様に対する思わぬ考え漏れや勘違いを防ぐことは、開発全体の質や開発速度を上げる観点で非常に有効です。

Haskellは、代数的データ型やパターンマッチを簡潔に記述できる言語的な特徴を持つため、仕様を簡潔に表現することに向いています。さらに、対話環境GHCiを用いて、自分の考えを試行錯誤的に確認できます。

自然言語等の仕様を、プログラミング言語を用いて表現・写経する過程は、単純ですが、対象への理解を深める上で、意外に大きな投資対効果があります。Haskellは、このような場合に強力なツールとなります。


## モデル確認

設計の初期段階において、自分の考えミスを抽出するために、設計の中核部分を簡単なモデルで表現して確認することは、開発全体の質や開発速度を上げる観点で非常に有効です。

前節の仕様理解の場合と同様に、設計の中核モデルを簡潔に記述する目的で、Haskellを用いることが出来る場合があります。

Haskellの代数的データ型とパターンマッチは、モデルの簡潔表現にもフィットする場合が多く、自分の考えを手早く確認することに有効に使用できます。

さらに、Haskellで記述したモデルを、[`QuickCheck`](https://hackage.haskell.org/package/QuickCheck)ライブラリなどによるランダムテストパターンを用いて簡易検査することにより、値の範囲や特性に対する考え不足を、容赦なく効率的に抽出できます。


## モデル探索

設計の初期段階において、モデルのパラメータなどについての設計空間を、試行錯誤しながら探索したい局面があります。

前節のモデル理解の場合と同様に、設計空間を探索する目的で、Haskellを用いることが出来る場合があります。

Haskellの代数的データ型とパターンマッチを用いてモデルを簡潔に記述できれば、系の大きさなどの多くのパラメータを振りながら、最適な設計値を探索することに活用できます。


## その他

思考フェーズでは、記述したプログラムの実行速度よりも、思考内容をコードで表現する速さや、試行錯誤的にコードの内容を確認・変更する速さの方が重要なことが有ります。

各々の人の思考特性によりますが、Haskellの代数的データ型とパターンマッチなどの言語的な特徴は、実行可能仕様書・実行可能思考表現として、思考を整理することに向いています。

以下のように、Haskellを用いて、簡潔に、素早く、手軽に、思考作業を支援・加速できます。

  * モデルなどの思考を、代数的データ型で直感的・簡潔に記述する
  * 処理をパターンマッチを用いて簡潔に記述する
  * 対話環境GHCiで、挙動と思考を手早く試行錯誤的に確認する

便利ですね。


------

おわりに
=======

この記事では、Haskellを業務でカジュアルに使う観点やヒントについて紹介しました。
「作業」や「思考」が必要な、よりたくさんの局面でHaskellを使用できます。

関数合成、部分適用、高階関数、多相関数などの言語的な特徴は、関数をボトムアップや対話的に、素早くかつ高品質に組み上げるのに便利です。代数的データ型などの言語的な特徴は、ある種の思考パターン（選択、非一様、入れ子など）をストレートに表現するのに便利です。対話環境GHCiは、試行錯誤的に作業や思考を進めるのに便利です。

Haskellに限らず、自分の思考特性にあったプログラミング言語を、業務を加速する日常的なツールとして備えておくことは有用です。

しかし、そもそもプログラミング言語の可能性・適用範囲は非常に広いものです。その適用範囲を、「業務」に狭めてしまう必要もありません。

プログラミング言語は、業務のみに限らず、日々の「思考」の支援・加速に広く使用できるものです。


以上、 Enjoy programming！
