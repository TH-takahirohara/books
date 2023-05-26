初めてのSQL 第3版

# 1章 背景情報
## SQL
### SQL文の分類
- SQLスキーマ文 create tableなど
- SQLデータ文 insertなど
- SQLトランザクション文
- SQLスキーマ文を使って作成されたデータベース要素はすべてデータディクショナリという特別なテーブルに格納される。メタデータ

# 2章 データベースの作成と設定
- sakilaサンプルデータベース
- dockerを使う場合([参考](https://qiita.com/okumurakengo/items/727d15e3ab2d22cdb1f8))
  - `docker container exec -it sakila mysql -uroot -proot -Dsakila`

## データ型
### 文字データ
#### 文字セット
- シングルバイト、マルチバイト文字セット
- MySQL 8 のデフォルトはutf8mb4

#### テキストデータ
- テキスト型
- ドキュメントを格納するようなケースでは、mediumtext型かlongtext型のどちらかを選択する

### 数値データ

### 時間データ
- 時間の長さに関するデータはtime型を使う

## 2.4 テーブルを作成する

## 2.5 データの挿入とテーブルの変更
- insert時は、auto_incrementの列はnullで追加すれば、次に利用可能な値を自動的に挿入してくれる

## 2.6 文も使い方次第

## 2.7 Sakilaデータベース

# 3章 入門：クエリ
