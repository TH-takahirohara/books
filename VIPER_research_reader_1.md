# VIPER研究読本I クリーンアーキテクチャ解説編

## 第4章 GitHubリポジトリを検索するサンプルコード

### Presenter

- Presenterが依存するRouter,Interactorは、Presenterの初期化時に引数として渡すStruct（サンプルではDependency構造体）のプロパティとして持たせる（依存性の注入）。
- Viewに処理の結果を伝えるために、Viewのプロトコルをプロパティとして保持する。
- Interactorの結果を保持するプロパティを用意し、自身でセットした際に結果をViewへ伝える。
- 画面遷移処理時は、Routerのメソッドを呼び出し、引数で関連情報を与える。

### ViewController

- displayDataプロパティに表示データの情報を保持させる。
  - サンプルだと検索結果などを入れてる。Presenterから実行させるsearchedなどのメソッド実行時には、displayDataで保持するデータを更新し、その後Viewの更新（サンプルではtableViewリロード）する。
  - tableViewのデリゲートメソッドでは、displayDataで保持するデータを参照させる。
- ViewControllerの初期化（AppDependenciesによるモジュール組み立て時に行われる）時に、presenterをinjectする。（サンプルではinjectしていたら、iOS13以降であれば、injectではなく引数で渡せるはず。）

### Router

- Routeは初期化時に、引数として対応する遷移元のViewControllerを渡す。この遷移元ViewControllerはプロパティとして保持しておく。
- 画面遷移時にAppDependenciesのメソッドを呼び出し、AppDependenciesが遷移先画面のモジュールの依存解決を行い、その結果の遷移先ViewControllerを取得する。
- 取得した遷移先ViewControllerに、遷移元ViewControllerからpushなどして遷移させる。
