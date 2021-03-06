新しいLinuxの教科書

# Chapter 01 Linuxを使ってみよう

# Chapter 02 シェルって何だろう?
## 2-1 シェルとコマンド
- カーネル：OSの中核となる部分であり、CPUやメモリなどのハードウェアを管理するとともに、コマンド実行を処理するプロセス管理も行っている。
- シェル：カーネルのインタフェースとなっているソフトウェア。ユーザーがLinuxを操作するには、基本的にシェルを通して行う。

## 2-2 プロンプト
- シェルが表示している`[username@hostname ~]$`のような部分をプロンプトという。
- 多くのLinuxドキュメントでは、一般ユーザでは$、スーパーユーザでは#でプロンプトを例示する。
- ログイン時に最初に起動されるシェルのことを、ログインシェルと呼ぶ。Linuxでは、明示的に指定しない限りログインシェルにはbashが利用される。

## 2-3 シェルの種類
- sh：最も古くから存在するシェル。シェルスクリプトを書く際にはshを利用するのが一般的。
- csh
- bash：shを基本として、機能を拡張したシェル。シェルスクリプトを書くのにも向いている。
- tcsh
- zsh：非常に多くの機能を持つ。

## 2-4 どのシェルを選ぶか
- bashを利用することを強くお勧めする。
- 一時的に別のシェルを使いたいときは、shやzshなどの別のシェルの起動に対応するコマンドを実行すれば起動する。一時的に起動したシェルは、ただのシェル（非ログインシェル）を元のシェルに重ねて起動しているような状態である。このシェルから抜けるためにはexitコマンドを実行すれば良い。

## 2-5 ターミナルとは
- ターミナルは、ユーザがコンピュータへ入出力する際に利用する専用のハードウェアのこと。ターミナルの入力装置にはキーボードが、出力装置には主にディスプレイなどが使われる。
- 現在は、物理的なターミナルをソフトウェアによって実装した、ターミナルエミュレータが利用される。単にターミナルと言えば、このターミナルエミュレータのことを指すことも多い。
- **ターミナルエミュレータとシェルは、全く違うソフトウェア**なので、混同しないように注意すべきである。ターミナルエミュレータは、入出力の画面を提供するだけのソフトウェアである。

# Chapter 03 シェルの便利な機能


# Chapter06 探す、調べる
## 6-1 ファイルを探す

findコマンド　ディレクトリツリーからファイルを探す
- ファイルを探す
`find <検索開始ディレクトリ> <検索条件> <アクション>`

- findコマンドの例
`$ find . -name file-1.txt -print`

- -name file-1.txtでファイル名を絞り込む。-printはパス名を表示するアクション。（アクションがない場合には-printが指定されたとみなされるので省略可。）


- ファイル名で探す（-name, -iname）
  - -name: ファイル名の大文字小文字を区別
  - -iname: 大文字小文字を区別しない
  - ファイル名の指定にはワイルドカードとして*と?を利用できる。
  - *は任意の文字列、?は任意の1文字に一致する。
  - -nameで*や?を利用する際には、シングルクオートを必ずつける。付けないと*などの記号がbashのパス名展開と解釈されてしまうため。


- ファイルの種類で探す（-type）

| 指定 | ファイル種別 |
| ---- | ---- |
| -type f | 通常ファイル |
| -type d | ディレクトリ |
| -type l | シンボリックリンク |

例）　カレントディレクトリの下にあるディレクトリを列挙
```
$ find . -type d -print
```

- 複数の検索条件の指定（-a）
  - 検索条件を-aで区切って並べることで、AND検索が可能
  - 例）
  ```
  $ find . -type f -a -name ‘*.txt’ -print
  ```

### locateコマンド　ファイル名データベースからファイルを探す
- ファイル名データベースがない場合もあるので、入っていなかったらインストールする。

ファイルを検索する
`locate [オプション] <検索パターン>`

- データベースは１日に１回更新されるので、たった今作成したファイルなどは検索できない。


- 検索パターンで大文字小文字を無視させる → -i または--ignore-caseオプション
`$ locate -i notes`

- ファイル名だけを検索対象にする → -bまたは--basenameオプション
  - 途中のパス名に関係なく、ファイルの名前そのものに一致したファイルだけが表示される
`$ locate -b python`

- 複数の検索パターンを指定することも可能。その際はOR検索となる。
`$ locate docs document`

- AND検索をさせたい場合は、-Aまたは--allオプションを利用する。
`$ locate -A bash doc`

## 6-2 コマンドの使い方を調べる

### --helpオプション
コマンド自身のヘルプメッセージを表示する。
```
$ cat --help
```

### manコマンド　指定したコマンドのマニュアルを表示する
```
$ man cat
```

- キーワードからマニュアルを探す
man -k <キーワード>

- セクションで調べる
man <セクション番号> <名前>

- 特定のマニュアルページがどのセクションに含まれているかを表示
$ man -wa crontab


## 6-3 コマンドを探す
- コマンド入力時に実行ファイルを探す場所を、環境変数$PATHで設定される。
- シェルが実際にどのファイルを実行するかを確認したいときはwhichコマンドを使う。
which [オプション] <コマンド名>

# Chapter 07 テキストエディタ

# Chapter 20 ソフトウェアパッケージ
- Linuxでソフトウェアをインストールするには主に以下の2つの方法が用いられる。
  - ソースコードを自分でコンパイルして、指定ディレクトリにコピーしてインストールする。
  - パッケージと呼ばれるコンパイル済みバイナリファイルのアーカイブを、パッケージ管理システムを利用してインストールする。
- 本章では現在のLinux運用管理の主流である、後者を解説する。

## 20-01 パッケージとリポジトリ
- パッケージ管理システムでは、パッケージを単位としてインストール・アンインストールの管理を行う。
- パッケージとは、ソフトウェアの実行ファイル・ドキュメントファイル・設定ファイル・インストール時に必要なスクリプトなどをまとめてアーカイブした1つのファイルのこと。
- 現在のLinuxで主流のパッケージファイル形式は、**rpmとdevの2種類に大別**される。
- パッケージファイルを集めて配布しているサイトを、リポジトリと呼ぶ。単に「ファイルの配布場所」を意味する。
- パッケージ管理システムは、リポジトリを指定することでパッケージ情報やパッケージファイルを取得し、インストールを行う。
- ディストリビューションごとにデフォルトで指定されrているリポジトリがある。このようなものを公式リポジトリと呼ぶ。

## 20-02 yumコマンド - パッケージ管理（CentOS）

## 20-03 aptによるパッケージ管理（Ubuntu）
- UbuntuなどのDebian系Linuxディストリビューションでは、**debというパッケージファイル形式が採用されている**。
- debファイルのインストールには、一般にはAPT（Advanced Packaging Tool）系コマンドが利用される。

### 基本的な使い方
```
apt-get [オプション] [コマンド] [パッケージ名...]
apt-cache [オプション] [コマンド] [パッケージ名...]
```

### パッケージのインストール
```
sudo apt-get install <パッケージ名>
```

### パッケージ間の依存性
- 多くのパッケージは、「先にインストールしておかないといけない別のパッケージ」を持つ。APT系コマンドは、**yumコマンドと同様に依存性の解決を自動的に行なってくれる**

### パッケージの削除
```
sudo apt-get remove <パッケージ名>
```
- 依存性のあるパッケージもAPTが同時に削除してくれる。
- removeコマンドで削除した場合、設定ファイルなどが一部残る。すべて削除する場合はpurgeコマンドを利用する。`sudo apt-get purge <パッケージ名>`

### パッケージを探す
- リポジトリからパッケージ名を検索するには、searchコマンドを使う。一般ユーザで実行できる。
```
apt-cache search <検索ワード>
```

### パッケージの情報表示
- パッケージの詳細情報の表示にはshowコマンドを使う。
```
apt-cache show aptitude
```
