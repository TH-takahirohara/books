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
- MySQL 8 のデフォルトはutfmb4

#### テキストデータ
- テキスト型
- ドキュメントを格納するようなケースでは、mediumtext型かlongtext型のどちらかを選択する
