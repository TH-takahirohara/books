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

# Chapter 4 MySQLを使ったシステム開発時の観点
## 4.1 トランザクションとロック
### トランザクション
- 分離レベル　通常はリードコミッテッド、もしくはリピータブルリードを使う。
- 大前提としてInnoDBは MVCC を用いている。
- リードコミッテッド：　クエリを発行した時点でコミットされているデータを読み込む。同じクエリを複数回実行した場合、最新のクエリの実行開始時点でコミットされたデータを読み込むことになる。
- リピータブルリード：　初回クエリを発行時点でコミットされたデータを読み込む。同じクエリを複数回実行した場合、初回の読み込み内容で結果セットが返される。
- 一貫性のあるスナップショット：　トランザクション開始時にコミットされているデータを取得する
- トランザクション分離レベルの違いによって起こりうる主要な現象は、以下の2つ。これらは同じデータについて二度以上のクエリを発行する際に問題になる。
  - ファジーリード（ノンリピータブルリード）
  - ファントムリード

### トランザクション実現の仕組み（InnoDB）
#### 行単位のロック
- 行単位のロックは主キーに対して設定される。走査した行すべてに対してロックが設定される。対象行を特定するクエリなどでは、インデックスを作成しロックの範囲を最小化して、それが使われるようにする必要がある。
- 共有ロック、排他ロック
- 以下のことを気にかけるとよい
  - 排他ロックが取得されるような場合には、インデックスが使われるようにしてロック範囲を最小化する
  - 可能であれば、デフォルトのREPEATABLE-READではなく、READ-COMMITTEDを使う

#### InnoDBのインデックス
- クラスター索引
- セカンダリインデックス

#### MVCC
- 原子性を担保するために、コミットが行われるまで変更前のデータを保存しておくためのInnoDB共有テーブルスペース内の領域をロールバックセグメントという。この領域に格納される変更前のデータのことをUNDOログと呼ぶ。
- 各分離度での参照を正しく行うために、ロールバックセグメントに複数のUNDOログを持っている。

### ロックタイムアウトとデッドロック
#### ロックタイムアウト
- 更新と更新がぶつかった場合には、後からきた更新がロック待ち状態になる。その際、ロックを待つ時間を設定できる。
- ロック待ちでタイムアウトがでた場合、ロールバックされるのはエラーが出たクエリだけ

#### デッドロック
- InnoDBの場合、デッドロックが起きた時には、すぐに検知しシステムに対して影響が少ない方のトランザクションをトランザクション開始時点までロールバックする。
- デッドロックの頻度を下げるためにいくつか考慮すべきことがある。

#### トランザクションべからず集
- オートコミット
  - 通常のアプリケーションロジックを実行するには、オートコミットはコミットの負荷が高すぎる
- ロングトランザクション
  - 大量処理を1つのトランザクションで行う
  - 何もしないトランザクション
  - トランザクション中に対話処理を入れる

## 4.2 データ型
### 文字列型
- 固定長文字列型（CHAR）
- 可変長文字列型（VARCHAR）
- ラージオブジェクト型（TEXT）　通常はMEDIUM TEXTもしくはLONG TEXTを利用する
- 文字列型は、まずVARCHAR型の利用を考え、収まらない場合はTEXT型の利用を考える。

### 数値型
- 整数（INT）
- 浮動小数点（FLOAT/DOUBLE）
- 固定小数点（NUMERIC/DECIMAL）　誤差が出ては困るような値に使う

#### 数値型のまとめ
- 整数：19桁までBIGINT、それより大きい場合はNUMERIC
- 小数を含む場合：誤差OKならDOUBLE、誤差NGならNUMERIC
- FLOATは大量に使う場合の領域削減の際のみ利用する。

### 日付時刻型
- 日付型（DATE）
- 時刻型（TIME）　時分秒を扱う
- 日付時刻型（DATETIME）
- 日付時刻型（TIMESTAMP）
- INTERVAL　表の定義には使えない。演算に利用する。

### バイナリ型
- データを「バイナリ列としてそのまま」格納する
- BINARY, VARBINARY型　BINARYの最大長は255byte, VARBINARYの最大長は65532byte
- バイナリラージオブジェクト（BLOB）型　VARBINARYを超える長さの場合に、mediumblob, longblobを利用すべき

## 4.3 文字コード
### 一般的な文字セットとエンコーディングの考え方
- 文字セット：　コンピュータ上で処理できる文字集合を定義したもの。文字に対する数値（文字コード）は特定していないので、これだけではコンピュータ内に格納することはできない。
- エンコーディング：　特定の文字セットをコンピュータ内にどう格納するかを定義したもの。
- ざっくりまとめ
  - JIS 2004やサロゲートペアが必要な場合は、クライアント側にutf8mb4を指定する
  - JIS 2004やサロゲートペアが必要ない場合は、クライアント側にutf8またはcp932を指定する

### MySQLでのキャラクタセットの指定

### トラブルのパターンと解消法
1. 宣言と内容が違う場合
2. 文字コードの変換時、変換先の定義がない場合
3. 文字コードの変換時、複数の文字が1つにまとめられるため、再度変換したときに1つにまとめられてしまう（ラウンドトリップ問題）
4. 見かけが同じ（もしくは似ている）文字に別コードが割り当てられているため、検索結果が期待したものにならない（重複コード化、異体字）

### トラブルシューティング
1. クライアント、サーバー（実際のテーブル定義）を確認する
  - クライアント文字コードの確認 -> show variable like 'char%'もしくはstatus
  - テーブル定義の確認 -> show create table テーブル名¥G
2. 実際にどのように格納されているか、hex()で16進数表示してみる
3. 0x数値を使い漢字コードを指定して、実際にテーブルに格納してみる

## 4.4 SQLの便利な機能
### DDL: データ定義
- DBオブジェクト削除時にif existsをつけることが可能。作成時にif not existsをつけることが可能。エラーが出ない。
- 照合順序（コレーション: Collation）　デフォルトだとutf8_general_ciというコレーションが設定されている。これは大文字・小文字を区別しないもの。区別するための方法が3つある。

### DML: データ操作
- MySQLでは、CHARもVARCHARも空白埋め比較セマンティクスで比較される。
- コメント拡張  /*! */
- ユーザ変数 @variablename
- ||は論理演算子のORを表す。（SQL標準では||は文字列連結の意味）

#### INSERT
- バルクINSERT
- INSERT IGNORE

#### UPDATE
- INSERT ... ON DUPLICATE UPDATE

#### SELECT
- SELECTする場合、必要となる列、必要となる行だけをSELECTすることを心がける。
- `SELECT *`, `SELECT count(*)` の使用は止める。

### DCL: データ管理
- CREATE USER, GRANT文
- rootのパスワードを忘れてしまった場合

### MySQL独自構文
- SELECT ... INTO OUTFILE
- LOAD DATA [LOCAL] INFILE
- FLUSHオプション

# Chapter 5 テスト・QA時の観点
## 5.1 ストアドルーチンの利用
- MySQLではストアドルーチン記述用の言語としてSQL/PSMを利用する。

### 最小限のSQL/PSM入門
- ストアドプロシジャ　call文で呼び出す
- ストアドファンクション　値をRETURNで戻せる。call文は使えず、SQL文の中で利用する。

### トリガ
- DBに対する操作を引き金として実行されるストアドルーチンをトリガという。
- BEFORE UPDATE で、エラーになるような操作でも操作自体が記録されるようになる。

#### ストアドプロシジャ
- ストアドプロシジャはSQL文に制御構造をつけた一連の手続き（プロシジャ）をデータベース側に格納（ストア）して実行するもの
- 登場当初の利点が減ってきているので、ストアドプロシジャが必要な場面も以前より減ってきている。

#### ストアドファンクション
- 各種組み込み関数と同様にSQL文の中で使えて1つの値を戻す関数を、SQL/PSMを使ってユーザが作成できる機能である。ユーザがクエリと一緒に利用したい独自関数を定義する場合に使う。
- ストアドプロシジャと比較して以下が違う
  - 引数はすべてINのみとなる。
  - 戻り値はヘッダで宣言した型のものを1つ戻す。戻り値はRETURNで戻す。結果セットは戻せない。
  - SQL文の中で利用する。

## 5.2 MySQLのログ
### エラーログ
- mysqlの稼働中に起こる様々な事柄を記録する。
- エラーログの重要度　ERROR, Warning, Note
- エラーログ出力のレベルは変えられる。オプションlog-warnings

### 一般クエリログ
- MySQLサーバがクライアントから受け付けた処理を詳細に書き出すログファイル
- MySQL5.1以降では、一般クエリログの出力は動的にオンラインでON/OFF可能となり、出力先をディスクだけでなくテーブルにも出力できるようになった。

### スロークエリログ
- MySQL特有のログで、処理に時間のかかったクエリを記録するもの。

### バイナリログ／リレーログ
- ストレージエンジンの種別に関係なく、MySQL本体が更新クエリを書き出したもの
- バイナリログはレプリケーションやデータベースリカバリ時のロールフォワードリカバリの入力として扱う。

### InnoDBログ
- InnoDBログはInnoストレージエンジンの更新データをWALで書き込むためのファイル

### QA・テスト時のログ利用シーン
- 何か調子が悪い場合や、構成ファイルのオプションを変更したら起動しなくなったような場合には、まずエラーログを確認する。
- 接続できない場合や、文字化けをする場合、またアプリとのやりとりを詳しく知る必要がでた場合などは、一般クエリログを利用する。
- クエリがとにかく遅い場合には、スロークエリログを利用する。

## 5.3 MySQLをモニタする
- 静的なものとして設定（my.iniまたはmy.cnf）、動的なものとして実際（GLOBAL/SESSION VARIABLES）、状態（GLOBAL/SESSION STATUS）があると考えられる。

### SHOW GLOBAL VARIABLESとSHOW GLOBAL STATUS
- MySQLサーバーの最大接続数設定と実際の接続を確認
- 一時テーブルがメモリに収まらずに書き出された数を確認
- ソート処理がメモリに収まらずにディスクに書き出された数を確認

### InnoDBモニタ
- InnoDB内部の状態を表示するためにSHOW [ENGINE] INNODB STATUSコマンドがある。これを定期的に出力するには、innodb_monitorというテーブルを作成する。
- InnoDBバッファプールのヒット率を確認
- 最後に起きた外部キーエラー／デッドロックの確認

### よく利用するSHOWコマンド
- `SHOW PROCESSLIST;`

### INFORMATION SCHEMA
- 情報スキーマはSQL標準に定義されているもので、データベース上に定義されたオブジェクト（テーブルなど）のメタ情報を取得する

## 5.4 プランとヒント
### 実行計画（アクセスプラン）
- テーブルスキャンとインデックススキャン
- 統計情報と実行計画の関係
- 実行計画の確認　EXPLAIN

### EXPLAIN
- type=ALLの時は、インデックスが使われていないので注意
- インデックススキャン　range, ref, const
- type=indexは、インデックスが全件読まれたことを示す。一般に負荷は高いので注意
- インデックス取捨の制御　FORCE INDEX, (USE INDEX,) IGNORE INDEX

### インデックスの種類と操作
- カバリングインデックス　インデックススキャンだけでクエリをまかなうこと
- 部分インデックス　text型のカラムにインデックスを作成する場合、先頭からの長さを指定する必要がある。基本的に可能な限り小さな値を指定する。

### 改善方法
#### キーの候補に挙がるようにする
- 文字列の比較
  - =の等価比較、likeの完全一致、前方一致はキーが使われる
- 日付時刻の比較　timestamp関数は日付型を日付時刻型に変換する
- データ型にあったデータで比較する　文字列と数値の比較などはしない

#### 統計情報を更新する
- 予期せぬタイミングでの統計情報の更新が起きないように設定する

#### テーブル定義を改善する
- 数値を入れる場合は数値型に

# Chapter 6 安定した開発 〜初期運用のために
## 6.1 堅牢な稼働のための設定
- 接続数の上限をむやみに大きくしない
- DNSへの問い合わせをしない
- 適切な権限を持ったユーザを作成する
- 各接続の負荷ピークを抑える
- 一時ディスク領域を十分にとる
- なるべくエラーを出す設定
- ファイル料を必要以上に増加させない
- 障害に備えるログ設定　必要なログをディスクに同期書き込みする
- バックアップを取得する
- ベンチマークを取る

## 6.2 障害対策の考え方
### データベースの障害
- 論理障害、物理障害

### 物理障害
1. データ格納先（ハードディスク）の障害
2. サーバーの障害
3. サーバー・ディスクが置かれた場所自体の障害

### データ格納先（ハードディスク）の障害
1. データ格納先をブラックボックスとしてデータベース側で対応する
  - バックアップ・リストア
  - レプリケーション
2. データベース側の処理を変えずに、格納先で対応する

### バックアップ・リストア
- データの複製として待避されたデータを「バックアップデータ」、このデータを取得することを「バックアップ」と呼ぶ。
- 待避したデータからDBのデータが利用できる状態まで回復することを「リストア」と呼ぶ。
- 次の観点がある
  1. ホットバックアップとコールドバックアップ　バックアップ時のDBの状態による分類（稼働中か停止中か）
  2. 論理バックアップと物理バックアップ　取得するバックアップデータの形式による分類（テキスト形式とバイナリ）
  3. フルバックアップと部分（増分・差分）バックアップ　バックアップ時の対象とそれによるデータ量で分類
- ロールフォワードリカバリ：　フルバックアップしたデータ + フルバックアップ取得時以降のすべてのバイナリログ　で任意の時点までリストアする
- レプリケーション：　障害が起こった時に複製として使えるデータベースを作成する方法（バックアップとは異なる）

### やってはいけないこと
- バックアップとして名前を挙げたファイルは、それぞれできるだけ離れた場所に置く。1つのディスクに置くようなことはやってはいけない。

## 6.3 バックアップ・リストア（基本編）
### コールドバックアップ、リストアの実際
- OSコマンドを使う

### ホットバックアップ（mysqldump）
- ロックにかかわるオプション
  - lock-tables, lock-all-tables, single-transaction
- ホットバックアップからのリストア
  - リストア時にmysqlコマンドラインツールを使うので、MySQLサーバーが起動していることが必要

### リカバリ
- 障害直前の状態までロールフォワードリカバリする必要がある場合は、sync_binlog=1を指定してバイナリログの書き込みを同期書き込みにする必要がある。
- mysqlbinlog

### さらに進んだ方法
- MEB（MySQL Enterprise Backup）の利用
- OSのスナップショット機能の利用
