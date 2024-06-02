# 1章 ジェイウォーク（信号無視）

適切に交差テーブルを利用しないことで発生する。
交差テーブル（intersection）を無視するため、ジェイウォーク（信号無視）と名付けられた。

## 例

`product`テーブルにお問い合わせ用のアカウントの`account_id`を複数設定するために、
`account_id`を複数カンマ区切りで繋ぎ合わせて文字列として格納する。

```SQL
CREATE TABLE product (
    product_id SERIAL PRIMARY KEY
    account_id VARCHAR(100)
)
```

## 何がまずいか

- `account_id`を使った検索や結合に正規表現などによる工夫が必要になる。
- インデックスを効果的に利用できないのでパフォーマンスが落ちる。
- `account_id`に複数のアカウントIDを入れる場合にサイズ上限に引っかかる可能性がある
- 集約クエリ（SUM、COUNT、AVGなど）が使えなくなる
- アカウントIDの追加や削除、更新ではアプリ側での文字列操作が必要になる
- バリデーションがアプリ側で必要。全く関係のない"banana"のような文字列が入る危険性がある。

## アンチパターンを用いても良い場合

カンマ区切りフォーマットのデータが必要で、各要素への個別アクセスが不要な場合

## 解決策

交差テーブルの作成
```SQL
CREATE TABLE Contacts (
    product_id BIGINT UNSIGNED NOT NULL,
    account_id BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (product_id, account_id),
    FOREIGN KEY product_id REFERENCES products(product_id),
    FOREIGN KEY account_id REFERENCES accounts(account_id)
)
```
- より柔軟にクエリを発行できる。
- インデックスを効果的に利用できる。


