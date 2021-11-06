プロになるためのデータベース技術入門

# Chapter 2 データベースの基本とMySQLの仕組み
## リレーショナルデータベースが必要な理由
### トランザクション（ACID特性を持つ）処理
- Atomicity(原子性)
- Consistency(一貫性)
- Isolation(隔離性)　ロック、MVCC
- Durability(持続性)　InnoDBログ

## データベースとディスクとメモリ
### ディスク関連の仕組み
- 同期I/O、非同期I/O
- 遅延書き込み、同期書き込み

### メモリ関連の仕組み
- 仮想メモリ、物理メモリ
- 仮想アドレス空間
- InnoDBではログファイルとデータファイルの組で、耐障害性を持っている

## 2.3 MySQLの内部構造（メモリ・ディスク）
### データベースの起動
- 起動・終了するMySQLサーバーのことを「インスタンス」という。通常1つのサーバーマシンに、1つのインスタンスを稼働させる。
- MySQLでは、1つのインスタンスに対して複数のデータベースを作成できる。スキーマ＝データベース

### クライアントツアー
- MySQL独自の仕組み
  - クエリキャッシュ
  - ストレージエンジン
- ストレージエンジン：　インデックスやデータの読み書き、バッファリングの方式などを、ストレージエンジンI/Fに沿ったものに差し替えることができる。
- InnoDBストレージエンジン：　クラッシュセーフで、ACIDに準拠した形でトランザクションをサポートしているストレージエンジン
- InnoDBバッファプール：　InnoDBテーブルスペースに対するディスクキャッシュ

### クライアントツアー2（同じクエリの実行）
- クエリキャッシュ：　SELECT文のテキストを、その検索結果と合わせて格納する仕組み。クエリのパース以降の処理がスキップされるので、大変高速

### クライアントツアー3（データの挿入・コミット）
- InnoDBテーブルスペースとInnoDBログファイルは、二人三脚でクラッシュセーフとACIDを成立させている
- バイナリログ

### クライアントツアー4（MyISAMからデータセレクト）
- 独自のバッファとして、キーキャッシュという仕組みがある。
- MyISAM型のテーブルは、frm（テーブル定義を格納）, MYD（データを格納）, MYI（インデックスを格納）ファイルからなる。

## 2.4 MySQLの内部構造（構成ファイル・プロセス／スレッド）
### MySQL起動時に読み取る値
- MySQLサーバーは、自身の起動時の設定等（利用するバイナリ群の場所、動作設定など）を3つの設定を元に知ることになる。
  1. バイナリコンパイル時のデフォルト値
  2. バイナリ起動時の構成ファイル（my.iniまたはmy.cnf）
  3. バイナリ起動時のオプション値
- 3つの中で、(2)バイナリ起動時の構成ファイル（my.iniまたはmy.cnf） に明示的に記述することがおすすめ
- 3つの設定をもとに設定された情報は、起動後には「サーバーシステム変数」に格納される。

#### 起動時の構成ファイル
- 構成ファイル内は以下のような形で記述する。
```
[グループ名]
変数名=値
```
- MySQLサーバー（インスタンス）は、通常は[mysqld]グループの設定を読み込むと考えて良い。

### プロセス／スレッド構成
#### マルチスレッド構成
- MySQLは1つのサーバープロセス（mysqld）と、そこから生成された複数のスレッド群で構成される。
- スレッドはプロセス内での実行単位

#### サービスとしての動作
- mySQLは通常のアプリケーションとしての起動以外にサービス（POSIXでいうデーモン）としての起動が可能

### ディレクトリ構成
- basedir： 通常はインストールディレクトリを指定する。そうすることで、MySQLは必要なバイナリを利用できるようになる。
- datadir： データベースファイルやログファイルを作成するディレクトリ
- tmpdir： 一時的なファイルを作成するディレクトリ

### ユーザスレッド生成時に起こること
- スレッド生成時の手順
  1. スレッドの生成・初期化
  2. セッション変数への値のコピー
  3. 必要に応じてメモリの確保・解放
- グローバル変数に対して、各クライアント接続の操作に影響するものをセッション変数という

### リソース（スレッド、テーブルハンドラ）のキャッシュ
- リソースの確保・解放の負荷を軽減するために、MySQLサーバー側では以下のOSリソースのキャッシュが可能
  - スレッドのキャッシュ
  - テーブルハンドラのキャッシュ

# Chapter 3 MySQLの操作
## 3.1 本体付属のユーティリティ一覧とmysqlコマンドの簡単な利用法
### 本体付属のユーティリティ
- mysqlコマンドラインツールなど

### mysqlコマンドラインツールの体験
- オートコミット　コマンドごとにコミットする。デフォルトではオンになっている。

## 3.2 管理・対話ユーティリティ
### mysql
- SHOW、SET

### mysqladmin
- MySQLデータベースを管理するためのコマンドを実行する