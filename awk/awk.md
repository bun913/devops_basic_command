# awkがプログラミング言語ということは知っている?なら言語仕様も言えるよね？

**この記事は私がzenn.devに掲載している内容の転機です**

https://zenn.dev/articles/ca06df3f783bc7/edit

タイトルは私が実際にシェル芸エンジニアに言われたことです。

awkがプログラミング言語だという認識自体、数ヶ月前にしたばかりですが、言語仕様とか考えたこともなかったです・・・

最近は PythonやPHPといったLL言語を使うことが多いのですが、ログ集計なんかにとてもお世話になっているので、LL言語の元祖awkがどんな言語なのか仕様を調べてみることにしました。

とりあえず、集計等でさっと使うレベルでは困らない様になったかと思います。

## awkの言語仕様

### パターンとアクション

awkはパターンとアクションの言語とのこと。

- パターン
	- もし〜だったら
- アクション
	- こうする

```bash
パターン {　アクション　}
```

awkの入力として渡したテキストを１行ずつ処理していきます。

例えば以下のテキストから、abcという行の値だけ取り出したいときは以下のようにします。

```bash
# sample.txt
abc
def
gfd
abc

# awk
awk '$0 == "abc" { print $0 }' sample.txt
```

$0は組み込み変数で、処理中のレコードそのものを指します。

ちなみに、パターンとアクションは省略可能です。
省略すると次のようになります。


```bash
# パターンを省略した場合には全てのレコードに対してアクションが行われる
echo "abc" | awk '{ print }'
> abc

# アクションが省略された場合にはパターンにマッチしたレコードを表示する
# 下記の場合 aもbもcも空文字でも0でもないため真に合致し全て表示される
echo "a b c" | awk '1'
> a b c

# パターンもアクションもない場合には何も表示されない
echo "abc" | awk ''
> 

# パターンもアクションもある場合
# ちなみ配列の添字は1からです
echo "a b c" | awk 'split($0, arr) {print arr[1]}'
> a
```

### 真偽値の取り扱い

- 数字の0 は偽となる
- 空の文字列は偽となる
- 上記以外は全て真となる

```bash
echo "hoge" | awk '{ print $0 == "hoge" }'
> 1

echo "hoge" | awk '{ print $0 != "hoge" }'
> 0
```

上記のように比較演算の結果は `0` `1` で返却される

### 代入

他言語のように変数と代入もあります。

また**組み込み変数の値も含めて破壊的な変更が可能**です。
（これかなりびっくりしました）

例えば `NF` という現在のレコード中のフィールド数を示す組み込み数へ代入します。

```bash
echo "a b c" | awk 'NF = 2 '

> a b
```

フィールド数が２に置き換わって、3個目のフィールドcが表示されません。

### 前処理と後処理

awkでは前処理・メイン処理・後処理とブロックに分けることができます。

```bash
BEGIN{
	#前処理　変数の宣言とか
}

{
	#メイン処理
}

{
	#後処理　メイン処理の結果を表示したり
}

```

例えば以下のテキストで値段の部分の合計を出力してみます。

```bash
# sample.txt
id	買ったもの	値段	数量
1	パン	100	3
2	牛乳	50	4
3	喧嘩	50	1

# 値段の合計
awk  ' BEGIN{ priceSum = 0 } NR>1 { priceSum += $3 } END{ print priceSum }
> 200

# NRは組み込み変数で Nomuber OF Lineの略です。現在の行（レコード）数ですね。ヘッダーが邪魔なのでパターンに NR > 1と指定して２行目以降を対象としています
```

### 条件分岐

当然if文もありますよ！！

```bash
awk ' NR>1 { if ($3 == 100){ print $1 "は100円" } else { print $1 "は100円じゃない" } }' sample.txt
```

### 四則演算

先程の例でしれっと足し算をしています。

特定のフィールド（値段）の四則演算だけでなく、特定のフィールド同士の演算もできます。

```bash

awk ' BEGIN{ totalPrice = 0 } NR > 1 { totalPrice += $3 * $4 } END{ print totalPrice } ' sample.txt 
> 550
```

※ サイン・コサインを求める関数もあったりします

### 文字列処理

awkが大得意らしい文字列処理です！

私が大好き配列もあります。以下ではarrという変数に配列を代入し、２番目のフィールドを利用しています。

```bash
# split.txt
1-1-0	test
1-2-0	test2
1-2-1	test3

# 最初のフィールドについて -で分割して最後のブロックだけ利用する
awk ' split($1, arr, "-") { print arr[3],$2 }' split.txt
```

```bash
0 test
0 test2
1 test3
```

文字列の抜き出し

```bash
echo "abc" |  awk '{ print substr($0, 2, 2) }'
> c
# 2文字目から２文字抜き出す
```

## まとめ

awkはすごい。

デフォルトのLinux環境で使えるので、わざわざ他言語でスクリプトを組む必要がありません。

これでログ集計スクリプトとかを組んだら大喜びされました！！

これからもシェルで簡単にできることはシェルでさっと終わらせる術を覚えていきたいと思います！！