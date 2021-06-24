SQL 第2版 ゼロから始めるデータベース操作

# 第0章 イントロダクション
macでのpostgresqlインストール
- [この記事](https://qiita.com/ksh-fthr/items/b86ba53f8f0bccfd7753)を参照

設定後のログイン
```
psql -Upostgres
```

DBの作成、psqlの終了
```
postgres=# CREATE DATABASE shop;
CREATE DATABASE
postgres=# \q
```

学習用データベースshopへのログイン
```
% psql -Upostgres -dshop
psql (13.1)
Type "help" for help.

shop=#
```
後の章ではこのDBでSQLを実行していく。

# 第1章 データベースとSQL
