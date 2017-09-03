---
title: Haskellの種(kind)について (Part 2)
headingBackgroundImage: ../../img/post-bg.jpg
headingDivClass: post-heading
subHeading: 種の仕組みとそれに付随する言語拡張について
postedBy: mizunashi-mana
date: September 3, 2017
...
---

Haskellには種(kind)という仕組みがあります。大雑把に言ってしまえば、「型の型」を実現する仕組みです。この仕組みについて、あまり情報が出回っていないようなので、解説記事を残しておこうと思います。なお、前編と後編に分かれていて、この記事は後編になります。前編は[こちら][part1-link]になります。

この記事は、[Ladder of Functional Programming](http://lambdaconf.us/downloads/documents/lambdaconf_slfp.pdf) ([日本語訳](http://qiita.com/lotz/items/0d68c8440d1f362d0c32))の**FIRE LUBLINE(ADVANCED BEGINNER)**を対象に、[Part 1][part1-link]の続きとして、種に付随するGHC言語拡張やパッケージを紹介するものです。

なお、特に断らない限り、対象としてGHC8系を設定しています。`stack`を使ってる方は`resolver`をLTS Haskell 8以降に設定しておくことを推奨します。

## 様々な種

### 型制約の種

[前回の記事][part1-link]では、種の基本的な仕組みを紹介しました。全てのデータ型は`*`という種を持っており、データ宣言は`*`の種を持つ型を作る型コンストラクタを定義するのでした:
```haskell
>>> newtype WrappedInt = WrappedInt Int
>>> :kind WrappedInt
WrappedInt :: *
>>> data Tag a = Tag
>>> :kind Tag
Tag :: * -> *
```

さて、Haskell標準には上のようなデータ型を表す`*`と、型コンストラクタを表す`k1 -> k2`という形の種しかありませんでした。GHCでは、他にもいくつか種を導入しています。今日は、その幾つかを紹介していきます。一つ目が、型制約を表す種`Constraint`です。この種を伴う仕組みは`ConstraintKinds`拡張により導入できます。

Haskellの型上には、データ型や型コンストラクタの他にも、型制約という登場人物がいました。型制約は名前の通り、型の制約が書けるようにするものでした。以下の関数をみてください:
```haskell
min :: Ord a => a -> a -> a
min x y = if x < y then x else y
```
この関数`min`は、型`a`が順序を持つ(`Ord a`のインスタンスである)という制約を満たしている時、二つの引数のうち小さい方を`Ord`のメソッド`<`を使用して返します。

型制約は、型クラスを使うことで作ることができました。例えば、ある型がデフォルトの値を持つという制約は、以下のように書けます:
```haskell
class HasDefault a where
  defaultValue :: a
```
この型クラスを使うことで、デフォルトの値を持つ型制約を満たしている型上では、`defaultValue`メソッドを使用することができるようになるのでした。例えば、以下のようにです:
```haskell
fromMaybe :: HasDefault a => Maybe a -> a
fromMaybe (Just x) = x
fromMaybe Nothing  = defaultValue
```

さて、GHC上では、型制約にも種が割り当てられています。見てみましょう:
```haskell
>>> :kind HasDefault
HasDefault :: * -> Constraint
```
`HasDefault`型クラスは、`*`の種を持つ型を受け取り、`Constraint`の種を持つ型制約を返します。実際に、データ型を適用して型制約にしてみましょう:
```haskell
>>> :kind HasDefault Bool
HasDefault Bool :: Constraint
```
適用結果は、ちゃんと`Constraint`の種を持っています。ここで、種の計算では型の計算は行われないことに注意してください！私たちは、`Bool`を`HasDefault`のインスタンスにしていないため、実際にはこの型制約は満たされません。型の計算を実際に行ってみましょう。上で定義した`fromMaybe`を実際に`Bool`型の値に使ってみます:
```haskell
>>> :type fromMaybe $ Just True

<interactive>:1:1: error:
    • No instance for (HasDefault Bool)
        arising from a use of ‘fromMaybe’
    • In the expression: fromMaybe $ Just True
```
「`HasDefault`は`Bool`に対してインスタンスを持っていない」と型エラーになっていることが分かります。これらの型計算は、もう少し直接的に確かめることもできます。次を見てください:
```haskell
>>> :set -XFlexibleContexts
>>> :type undefined :: HasDefault Bool => Bool

<interactive>:1:1: error:
    No instance for (HasDefault Bool) arising from a use of ‘it’
```
このように、型の計算だけを行わせる場合、`undefined`を使用するのが便利です。Haskell標準では、型制約はあまり柔軟には書けません。そこから、具体的な型を伴う上の制約も書けないため、`FlexibleContexts`拡張を使用することで書けるようにしています。

さて、以上の型表記で登場する、`=>`というものは、左で指定された型制約を満たしているならば右で指定された型付けの関数になる、という意味を持っています。つまり、型上の演算子として考えるなら、`(=>) :: Constraint -> * -> *`という種になります。なので、例えば以下のような型表記は、種の辻褄が合わなくなります:
```haskell
>>> :kind Int => Bool

<interactive>:1:1: error:
    • Expected a constraint, but ‘Int’ has kind ‘*’
    • In the type ‘Int => Bool’
>>> :kind HasDefault => Bool

<interactive>:1:1: error:
    • Expecting one more argument to ‘HasDefault’
      Expected a constraint, but ‘HasDefault’ has kind ‘* -> Constraint’
    • In the type ‘HasDefault => Bool’
```
`kind`コマンドによって、種が合わないとエラーになっていることが分かります。残念ながら、`=>`は実際には型演算子ではなく、`(=>)`というように型関数として扱うことはできません。ですが、それは表記上の問題であり、確かに種の計算の際、型制約を受け取る型演算子として、`Constraint`の種を持つか検査が行われていることは分かるでしょう。

さて、今までは型制約を表す種`Constraint`について、GHCi上で色々試しながら見てきました。型制約種に関しての雰囲気は分かってもらえたと思います。型制約種`Constraint`をもう少し詳しく見ていきましょう。以下を見てください:
```haskell
>>> data SimpleData a = SimpleData a
>>> :kind SimpleData
SimpleData :: * -> *
>>> class SimpleClass a where simpleMethod :: a -> a
>>> :kind SimpleClass
SimpleClass :: * -> Constraint
```
データ宣言と型クラス宣言を並べてみました。この二つはよく似ています。

* データ宣言は、`*`という種の型になるような型コンストラクタ`SimpleData :: * -> *`を作り、`SimpleData :: a -> SimpleData`という値コンストラクタを作ります。
* 型クラス宣言は、`Constraint`という種の型制約になるような`SimpleClass :: * -> Constraint`を作り、`simpleMethod :: SimpleClass a => a -> a`というような関数を作ります。

作られるものがそれぞれ違いますが、両方型の世界に一つの型関数、値の世界に関数を作るわけです。型クラスの方をデータ宣言に合わせるとしたら、型クラスは型制約コンストラクタとメソッドを作るものと言えるかもしれません。型の世界だけでの話なら、データ宣言は型コンストラクタを、型クラスは型制約コンストラクタを単に作るだけの構文ということになります。ではここで、データ型と型制約の対比を表にしてみましょう。

| 種 | 表現される型 | 定義方法 |
| --- | --- | --- |
| `*` | データ型 | データ宣言(`data C a = ...`) |
| `Constraint` | 型制約 | 型クラス宣言(`class C a where ...`) |

両者はそれぞれ特別な意味を与えられています。実際、データ型が値を持ち[^notice-data-type]それによって型注釈が書けるように、型制約は`=>`というような特別に型制約を計算するような、型上の構文を持っています。ですが、逆に言えば特別なのはそれだけで、それ以外に両者の違いはありません。例えば、`Proxy`型に型制約や型制約コンストラクタ(型クラス)を渡すこともできます:
```haskell
>>> import Data.Proxy
>>> :kind Proxy
Proxy :: k -> *
>>> :type Proxy :: Proxy (Monoid Bool)
Proxy :: Proxy (Monoid Bool) :: Proxy (Monoid Bool)
>>> :type Proxy :: Proxy Monoid
Proxy :: Proxy Monoid :: Proxy (Monoid Bool)
```
`=>`を使用していないので、型制約の計算は行われないことに注意してください！`Proxy`型は種多相化されており、どんな種の型も受け取れるのでした。

* `Proxy :: Proxy (Monoid Bool)`の場合は、`Proxy :: Constraint -> *`
* `Proxy :: Proxy Monoid`の場合は、`Proxy :: (* -> Constraint) -> *`

というようにそれぞれ`Proxy`型コンストラクタが特殊化されます。では、この`Proxy`で型制約を受け取り、型制約の計算だけを行うような関数を作って使って見ましょう。その関数は、以下のように作ることができます:
```haskell
>>> :set -XConstraintKinds
>>> f :: a => Proxy a -> (); f _ = ()
>>> -- この段階では、まだ型制約計算されない!
>>> Proxy :: Proxy (Monoid Bool)
Proxy
>>> -- 型制約計算をする関数に適用
>>> f (Proxy :: Proxy (Monoid Bool))

<interactive>:24:1: error:
    • No instance for (Monoid Bool) arising from a use of ‘f’
    • In the expression: f (Proxy :: Proxy (Monoid Bool))
      In an equation for ‘it’: it = f (Proxy :: Proxy (Monoid Bool))
>>> f (Proxy :: Proxy (Monoid String))
()
```
やっと、`ConstraintKinds`拡張の登場です。`ConstraintKinds`拡張は、型制約に関するHaskell標準の制限を幾つか取り払う拡張です。どのようなことが可能になるかは後で紹介するとして、今は上の関数の使い方に注目しましょう。このように、型制約は`Proxy`型で持ち回し`=>`で任意のタイミングで型制約計算を行うといったことも可能です。面白いですね。

[^notice-data-type]: 型コンストラクタは値を持てないことに注意してください！何らかの値を持つ型は全て`*`という種を持つものになっており、例えば`Maybe :: * -> *`という型コンストラクタはそれだけでは値を持たず、`Maybe Int`など型を一つ渡して初めて値を持つような型になるのでした。

上の`Proxy`を使った例から明らかですが、もちろんデータ宣言時に種注釈を使うことで型制約を受け取るようなデータ型を作ることもできるわけです。そして、型制約を受け取るような型クラスもです。次の例を見てください:
```haskell
>>> :set -XKindSignatures
>>> :set -XFlexibleInstances
>>> import GHC.Exts (Constraint)
>>> class AConstraint (c :: Constraint)
>>> :kind AConstraint
AConstraint :: Constraint -> Constraint
>>> instance AConstraint (Monad Maybe)
```
`FlexibleInstances`拡張は、`FlexibleContexts`拡張と同じような拡張で、`FlexibleContexts`は型制約の書き方の制限を、`FlexibleInstances`拡張はインスタンスの書き方の制限をそれぞれ取り払う拡張です。上のようにすれば、型制約の分類分けすらすることができるようになります。

最後に`ConstraintKinds`拡張をきちんと紹介しておきましょう。`ConstraintKinds`拡張は、次のようなことを可能にしてくれる拡張です。

* 型エイリアスと同じ構文で、型制約コンストラクタのエイリアスを書くことができるようになる。

    つまり、次のようなことが可能になります:
    ```haskell
    {-# LANGUAGE ConstraintKinds #-}

    -- 型制約のエイリアス
    type MonMonad m a = (Monoid (m a), Monad m)

    -- 型制約コンストラクタのエイリアス
    type Mappable = Functor
    ```


* 型制約種`Constraint`を持つ型を、型制約として使用できるようにする。

    こちらは、あまり実感が湧かないかもしれません。デフォルトで、GHCでは型クラスなどを型制約として扱う、つまり特に問題なく`=>`に渡すことができます。ですが、`Constraint`の種を持つ型制約変数などを渡すことはできません:
    ```haskell
    >>> import Data.Proxy
    >>> -- 型クラスを型制約として使っているため、問題ない
    >>> :type undefined :: Monad m => m a
    undefined :: Monad m => m a :: Monad m => m a
    >>> -- 型制約変数は、型制約として扱えない
    >>> :type undefined :: a => Proxy a

    <interactive>:1:14: error:
        • Illegal constraint: a (Use ConstraintKinds to permit this)
        • In an expression type signature: a => Proxy a
          In the expression: undefined :: a => Proxy a
    >>> :set -XConstraintKinds
    >>> -- 型制約種を持つ変数なら、型制約として扱えるようになる
    >>> :type undefined :: a => Proxy a
    undefined :: a => Proxy a :: a => Proxy a
    ```


### 種への昇格

### unlifted typesとlifted types

## advanced topics

### type in type

### もう一つの型の分類

### Levity polymorphism

### トップレベル種注釈

## 参考文献

* [Haskell 2010 Language Report](https://www.haskell.org/onlinereport/haskell2010/haskell.html): Haskell2010の仕様書です。主に標準の仕組みを紹介する際に参照しました。
    - [4.1 Overview of Types and Classes](https://www.haskell.org/onlinereport/haskell2010/haskellch4.html#x10-630004.1): 標準の型システムや型制約について、書かれています。
* [GHC 8.2.1 Users Guide](https://downloads.haskell.org/~ghc/8.2.1/docs/html/users_guide/): 主な種に関する参考資料としてとGHC拡張についての資料として参考にしました。
    - [9.11 Kind polymorphism and Type-in-Type](https://downloads.haskell.org/~ghc/8.2.1/docs/html/users_guide/glasgow_exts.html#kind-polymorphism-and-type-in-type): GHCにおいての種推論などの、種に関することが総括してあります。
    - [9.14 Constraints in types](https://downloads.haskell.org/~ghc/8.2.1/docs/html/users_guide/glasgow_exts.html#constraints-in-types): 型制約に関する、GHC上のいくつかの話題が書かれています。
* [GHC Wiki - Commentary/Compiler/Kinds](https://ghc.haskell.org/trac/ghc/wiki/Commentary/Compiler/Kinds): この記事のストーリーを決める際に参照しました。

[part1-link]: 10-about-kind-system-part1.html
