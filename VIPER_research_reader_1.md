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
