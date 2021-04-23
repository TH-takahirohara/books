# VIPER研究読本I クリーンアーキテクチャ解説編

## 第4章 GitHubリポジトリを検索するサンプルコード

### Presenter

- Presenterが依存するRouter,Interactorは、Presenterの初期化時に引数として渡すStruct（サンプルではDependency構造体）のプロパティとして持たせる（依存性の注入）。
- Viewに処理の結果を伝えるために、Viewのプロトコルをプロパティとして保持する。
- Interactorの出力結果を保持するプロパティを用意し、Presenter自身でこのプロパティに値をセットした際に結果をViewへ伝える。
- 画面遷移処理時は、Routerのメソッドを呼び出し、引数で関連情報を与える。

### ViewController

- displayDataプロパティに表示データの情報を保持させる。
  - サンプルではdisplayData（のプロパティ）に検索結果などを保持させている。ViewControllerが持つメソッド（Presenterからプロトコル経由で実行させる。サンプルではsearchedなど）内の処理において、displayDataで保持するデータをメソッドの引数に渡した値で更新し、その後Viewを更新（サンプルではtableViewリロード）する。
  - tableViewのデリゲートメソッドでは、displayDataで保持するデータを参照させる。（上記のリロードで反映される事になる。）
- ViewControllerの初期化（AppDependenciesによるモジュール組み立て時に行われる）時に、Presenterをinjectする。（サンプルではinjectしていたが、iOS13以降であれば、ViewControllerのinitiallizerの引数として渡せるはず。）

### Router

- Routerの初期化時に、対応する遷移元のViewControllerを引数として渡す。この遷移元ViewControllerはRouterのプロパティとして保持しておく。
- 画面遷移時にAppDependenciesのメソッドを呼び出すことで、遷移先画面のモジュールの依存解決を行い、その結果として遷移先ViewControllerを取得する。
- 取得した遷移先ViewControllerに、遷移元ViewControllerからpushなどして遷移させる。

## 第5章 GitHubリポジトリを検索するサンプルコードのテストコード

### Presenterのテスト

- Presenterのロジックに対応するメソッド（サンプルではsearch、selectなど）内の処理は、基本的にdependencyにプロパティとして持たせているinteractorやrouterのメソッド実行のみとなっているので、それらメソッド（interactorで言えばexecute）の出力として自前で用意したテストデータをセットし、それをうまく検証させる感じで実装する。

### Routerのテスト

- RouterTest用クラスのセットアップ時に、テスト用のappDependenciesとテスト用のViewControllerを引数に渡す形で、テストに使うrouterを作成する。
- routerの画面遷移メソッド（サンプルではpresentDetailなど）を実行した後、表示されるViewController（サンプルではnavigationControllerを使ってpushされる）が期待したViewConrollerと同じかどうかを検証する。
  - サンプルの検証例： XCTAssertTrue(pushedViewController is AppTestDependencies.TestDouble.DetailViewController)

### Interactorのテスト

- Interactorのテストコードではテストダブルは使わない。
- サンプルでは、interactor.executeの引数に渡したクロージャ内で、case successの中で取得した値が所望のものになってるか検証させている。

## 第6章 クリーンアーキテクチャについて

### 依存関係逆転の原則

- 依存関係逆転の原則のやり方
  - 具象型があるモジュールに抽象型を配置せず所有権の逆転をさせる
- 依存関係逆転の原則ではないもの
  - 抽象型と具象型が同じモジュールにあればその役割は「情報隠蔽」
- 依存関係逆転の原則の長所
  - 円の外側にあるデータベースのモジュールを内部から抽象型を仲介して取り出せる
  - ＊ モジュール間の依存関係を逆転することでお互いに依存していても解決できる

### 単一責任原則

- 単一責任原則は2つある
  - SOLID原則でない単一責任原則
    - 『どのモジュールもたった一つのことだけを行うべき』（異なる理由によって変更される処理は、別々のモジュールに分割する）
  - SOLID原則の単一責任原則
    - 『モジュールはたった一つのアクターに対して責務を負うべき』（同一モジュールにおいて異なるチームが利用するコードがあった場合、それらのコードを分割する）
- 本書のVIPERのUseCaseについては、前者の単一責任原則を強く製薬としている。
