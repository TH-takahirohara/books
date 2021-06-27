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

## 1-1 データベースとは何か
- 大量の情報を保存し、コンピュータから効率よくアクセスできるように加工したデータの集まりのことを「データベース」と呼ぶ。
- データベースを管理するコンピュータシステムのことを、「データベース管理システム（DBMS）」と呼ぶ。
- DBMSを使うことにより、大量のデータを多人数でも安全かつ簡単に扱える。
- 本書では「リレーショナルデータベース（RDB）」を「SQL」という専門言語で操作する方法を学ぶ。RDBは「リレーショナルデータベース管理システム（RDBMS）」で管理する。

## 1-2 データベースの構成
