# JQコマンドの使い方

## 事前準備

Macの場合Homebrewでインストール可能

```bash
brew install jq
```

## 基本的な使い方

### 全データを表示

```bash
# パイプで受け渡したり
cat 01_github_sample.json | jq '.'
# 普通に入力から受け取ったrり
jq '.' < 01_github_sample.json
```

ここでの `.` はルートの `{}` や `[]` を表す

### 配列の特定要素の取得

```bash
# 最初の要素の取得
jq '.[0]' < 01_github_sample.json
```

### 連想配列の特定要素を取得

```bash
# 最初の要素のnode_idを取得
jq '.[0].node_id' < 01_github_sample.json
# ちなみにjqコマンドのパイプオペレーター(フィルタ)で受け渡しも可能
jq '.[0] | .node_id' < 01_github_sample.json
```

### 全ての配列から特定の要素を抜き出し

```bash
# 配列中の連想配列からnode_idを抽出
jq '.[] | .node_id' < 01_github_sample.json
```

### JSONのフォーマット

フィールドの要素を加工して新しいJSONを出力

```bash
# node_idをid, commit.auhro.nameをauthor_nameとして出力
jq '[ .[] | {id:.node_id, commiter_name: .commit.author.name} ]' < 01_github_sample.json
```

## 少し応用

### 集計

```bash
# 各要素のpriceの合計
jq '[ .[] | .price ] | add' < 02_calc_sample.json
```

### 制御構文

```bash
# now_saleがtrueの要素だけ抽出
jq '.[] | select(.now_sale)' < 02_calc_sample.json
```

```bash
# itemがパンから始まる要素だけ抽出
jq '.[] | select(.item | startswith("パン"))' < 02_calc_sample.json
# パンを含む要素を抽出
jq '.[] | select(.item | contains("パン"))' < 02_calc_sample.json
```

複合条件式

```bash
# パンから始まってnow_saleがtrueの要素を抽出
jq '[ .[] | select( (.now_sale) and ( .item | startswith("パン") ) ) ]' < 02_calc_sample.json
# 米が含まれる　または priceが100円以下の要素を抽出
jq '[ .[] | select( (.price <= 100) or (.item | contains("米")) ) ]' < 02_calc_sample.json
```

上記は複雑に見えるけど
`select` で条件に合致するものだけフィルターするよ！と宣言して

`(now_saleがTrue)　かつ (itemがパンで始まる)`　と指定している

### from_entriesの使い方

ここの表題をなんてすれば良いか悩んだ・・・

例えば以下の内容↓を

```bash
[
  {
    "item": "パン1",
    "price": 100,
    "now_sale": true
  },
  {
    "item": "パン2",
    "price": 200,
    "now_sale": true
  },
  {
    "item": "パン3",
    "price": 300,
    "now_sale": false
  }
]
```

以下のような形で出力したい場合

```bash
[
    {
        "パン1": 100,
        "パン2": 200,
        "パン3": 300
    }
]
```

from_entriesに キー名を "key", バリューを "value"という連想配列で渡してあげるとよい

```bash
jq '[ .[] | { key: .item, value: .price}] | from_entries ' < 02_calc_sample.json
```

という非常に便利な機能が使えます

