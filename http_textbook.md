HTTPの教科書

## 2.7 持続的接続で通信料を節約
- HTTPの初期のバージョンでは、HTTPによる通信を1回行うごとにTCPによる接続と切断を行う必要があった。
- HTTP/1.1と一部のHTTP/1.0では、**持続的接続**という方法が考え出された。これは、どちらかが明示的に接続を切断しない限り、TCP接続を繋げっぱなしにしておくということ。
- これにより、TCPコネクションの接続と切断を繰り返すオーバーヘッドを減らせるので、その分Webページの表示が速くなる。
- これは、複数のリクエストを発行できる**パイプライン化**を可能にする。これにより、複数のリクエストを並行して発行できるので、1つ1つのレスポンスを待つ必要がなくなった。

## 2.8 Cookieを使った状態管理
- Cookieは、リクエストとレスポンスにCookieという情報を載せることによって、クライアントの状態を把握するための仕組み。
- Cookieは、サーバーからのレスポンスで送られたSet-Cookieというヘッダーフィールドによって、Cookieをクライアントに保存してもらうように指示を出す。次にクライアントが同じサーバーにリクエストを出す際には、クライアントが自動的に Cookieの値を入れて送信する。
- サーバーはクライアントが送ってきた Cookieを見ることで、どのクライアントからのアクセスなのかをチェックし、サーバー上の記録を確かめることで、以前の状態を知ることができる。

# 第3章 HTTPの情報はHTTPメッセージにある

## 3.1 HTTPメッセージ
- HTTPでやり取りされる情報はHTTPメッセージと呼ばれていて、リクエスト側のHTTPメッセージをリクエストメッセージ、レスポンス側をレスポンスメッセージと呼ぶ。
- HTTPメッセージを大きく分けると、メッセージヘッダーとメッセージボディから構成されて、空行が区切りになる。

## 3.2 リクエストメッセージとレスポンスメッセージの構造
- メッセージヘッダーの中身は、次のデータで構成される。
  - リクエストライン
  - ステータスライン
  - ヘッダーフィールド
    - 一般ヘッダーフィールド、リクエストヘッダーフィールド、レスポンスヘッダーフィールド、エンティティヘッダーフィールドの4種類
  - その他

## 3.3 エンコーディングで転送効率を上げる
- HTTPでデータを転送する場合、転送の際にエンコーディングを施すことで転送効率を上げることができる。
- メッセージ
- エンティティ
- コンテンツコーディング: エンティティに適用するエンコーディングのことで、エンティティの情報を保ったまま圧縮する。
- チャンク転送コーディング: エンティティを分割してから送信すること

## 3.4 複数のデータを送れるマルチパート
- マルチパート: MIMEの拡張仕様にある、複数の異なる種類のデータを格納する方式
  - MIME: メールでテキスト、画像、動画といった複数の異なるデータを扱うための仕組み
- HTTPもマルチパートに対応していて、1つのメッセージボディの中に複数のエンティティを含めて送ることができる。主に画像やテキストファイルなどのファイルアップロードの際に使われる。
- 詳細はRFC2046も参考のこと

## 3.5 一部分だけ貰えるレンジリクエスト
- 範囲を指定してリクエストを行うことを、**レンジリクエスト**と呼ぶ。
  - レジューム機能（以前ダウンロードを中断した箇所からダウンロードを再開できる）を実現する上で、エンティティの範囲を指定してDLを行う必要があり、その時などに使われる。
- レンジリクエストを行う場合、Rangeヘッダーフィールドを使って、リソースのバイトレンジを指定する。
- レンジリクエストに対するレスポンスは、ステータスコード206 Partial Contentが返される。

## 3.6 最適なコンテンツを返すコンテンツネゴシエーション
- 同じコンテンツだが、複数のページを持つWebページ（例えば、英語版と日本語版）について、クライアントに最も適したリソースを提供するための仕組みをコンテンツネゴシエーションと呼ぶ。
- コンテンツネゴシエーションでは、提供するリソースを言語や文字セット、エンコーディング方式などを基準に判断している。

# 第4章 結果を伝えるHTTPステータスコード

## 4.1 ステータスコードはサーバーからのリクエスト結果を伝える
- クライアントからサーバーに対してリクエストを送信したとき、その結果がどうだったのかということを伝えるのがステータスコードの役目
- ステータスコードは3桁の数値と説明で表す。（例：200 OK）
- 数値の最初の1桁でレスポンスのクラスを指定する。レスポンスのクラスは次の5つが定義されている。
  - 1XX: Informational リクエストが受け付けられて処理中
  - 2XX: Success リクエストは正常に処理を完了した
  - 3XX: Redirection リクエストが完了するには追加動作が必要
  - 4XX: Client Error サーバーはリクエストを理解できなかった
  - 5XX: Server Error サーバーはリクエストの処理を失敗した

## 4.2 2XX 成功 (Success)
- 200 OK
- 204 No Content
  - レスポンスにエンティティボディを含まない。
  - クライアントからサーバーに情報を送るだけでよく、クライアントに対して新たな情報を送る必要がない場合に使われる。
- 206 Partial Content

## 4.3 3XX リダイレクト (Redirection)
- 301 Moved Permantently
  - リクエストされたリソースには新しいURIが割り当てられているので、今後はそのリソースを参照するURIを使用すべきであるということを表す。
- 302 Found
  - リクエストされたリソースには新しいURIが割り当てられているので、そのURIを参照してほしいということを表す。ただし、移動はあくまで一時的なものである。
- 303 See Other
  - このレスポンスは、リクエストに対するリソースは別のURIにあるので、GETメソッドを使用して取得すべきであるということを表している。リダイレクト先をGETメソッドで取得するということが明確になっている点が302と意味が異なる。
- 304 Not Modified
- 307 Temporary Redirect

## 4.4 4XX クライアントエラー (Client Error)
- 400 Bad Request
  - リクエストの構文が間違っていることを表す。ブラウザはこれを200 OKと同様に扱う。
- 401 Unauthorized
  - 送信したリクエストにはHTTP認証の認証情報が必要であることを表す。すでに1度リクエストが行われている場合には、ユーザー認証に失敗したことを表す。
  - ブラウザで最初の401レスポンスを受け取った場合には、認証のためのダイアログが表示される。
- 403 Forbidden
  - リクエストされたリソースへのアクセスが拒否されたことを表す。
- 404 Not Found
  - リクエストしたリソースがサーバー上にないことを表す。

## 4.5 5XX サーバーエラー (Server Error)
- 500 Internal Server Error
  - サーバーでリクエストを実行する際にエラーが起きたことを表す。
- 503 Service Unavailable
  - サーバーが一時的な過負荷かメンテナンスのため、現在リクエストを処理することができないことを表す。

# 第5章 HTTPと連携するWebサーバー

## 5.1 1台で複数ドメインを実現するバーチャルホスト
- バーチャルホストの機能を使うと、物理的にはサーバーが 1台でも、仮想的に複数台あるかのように扱うことができる。
- 同じIPアドレスで、異なるホスト名やドメイン名を持った複数のWebサイトが稼動しているバーチャルホストの仕組みがあるため、HTTP リクエストを送る場合には、ホスト名やドメイン名を完全に含んだURIの指定か、Host ヘッダーフィールドでの指定が必ず必要になってくる。

## 5.2 通信を中継するプログラム：プロキシ、ゲートウェイ、トンネル

### プロキシ
- サーバーとクライアントの両方の役割をする中継プログラムで、クライアントからのリクエストをサーバーに転送し、サーバーからのレスポンスをクライアントに転送する。
- リソース本体を持ったサーバーのことを、オリジンサーバーと呼ぶ。
- プロキシを中継する際には、Viaヘッダーフィールドに経由したホストの情報を追加していく。
  - 例： proxy1, proxy2を経由したら、Via: proxy2, proxy1となる。
- プロキシサーバーを使用する理由としては、キャッシュを使ってネットワーク帯域などを効率よく使用することや、組織内で特定Webサイトに対するアクセス制限や、アクセスログを取得するといったポリシーを徹底させる目的として使用することもできる。
- プロキシは2つの基準で分類される。
  - キャッシュするかしないか　キャッシュするプロキシは**キャッシングプロキシ**と呼ぶ。
  - メッセージを変更するかしないか　中継時にメッセージに何ら変更を加えないタイプのプロキシを**透過型プロキシ**と呼ぶ

### ゲートウェイ
- 他のサーバーを中継するサーバーで、クライアントから受け取ったリクエストを、リソースを持っているサーバーであるかのように受け取る。
- ゲートウェイは、その先にあるサーバーがHTTPサーバー以外のサービスを提供するサーバーとなる。ゲートウェイを使うことで、HTTPリクエストによって、別のプロトコルを動作させることができる。

### トンネル
- 2つの離れたクライアントとサーバーの間で中継することで、2台の接続を取り持つ中継プログラム
- トンネルを使うことで、離れたサーバーと安全に通信させることができる。

## 5.3 リソースを保管するキャッシュ
- キャッシュ(Cache)は、プロキシサーバーやクライアントのローカルディスクに保存されたリソースのコピーのことを指す。キャッシュを使うことで、リソースを持ったサーバーへのアクセスを減らすことができるため、通信量や通信時間などを節約することができる。
- キャッシュサーバーというキャッシングプロキシの一種があり、転送時にリソースのコピーを保存する。キャッシュサーバーのメリットは、同じデータを何度もオリジンサーバーから転送する必要がなくなり、サーバー側が同じリクエストを何度も処理しなくて済むようになる。
- キャッシュには有効期限がある。
- キャッシュサーバーだけでなく、クライアントが使用するブラウザでもキャッシュを持つことができる。

# 第6章 HTTPヘッダー

## 6.1 HTTPメッセージヘッダー
- HTTPプロトコルのリクエストとレスポンスには、必ずメッセージヘッダーがあり、クライアントやサーバーの処理に必要な重要情報はほとんどこのメッセージヘッダーにある。
- メッセージヘッダーの要素の中で、最も多様な情報を持つのがHTTPヘッダーフィールド

## 6.2 HTTPヘッダーフィールド
- HTTPヘッダーフィールドは、メッセージボディのサイズや、使用している言語、認証情報などをブラウザやサーバーに提供するために使用される。

### HTTPヘッダーフィールドの構造
- HTTP ヘッダーフィールドは、ヘッダーフィールド名とフィールド値から構成されていて、コロンで区切られている。

### 4種類のHTTP ヘッダーフィールド
- HTTPヘッダーフィールドは、その用途に合わせて次の4種類に分類される。
  - 一般ヘッダーフィールド
  - リクエストヘッダーフィールド
  - レスポンスヘッダーフィールド
  - エンティティヘッダーフィールド

- HTTP/1.1ヘッダーフィールドには47種類ある。
- HTTP/1.1以外のヘッダーフィールドもある。（例えばCookieなど）非標準ヘッダーフィールドはRFC4229 HTTP Header Field Registrationsにまとめられている。

### エンドトゥエンドヘッダーとホップバイホップヘッダー
- HTTPヘッダーフィールドは、キャッシュと非キャッシュプロキシの振る舞いを定義する目的のために2つのカテゴリに分類されている。
  - エンドトゥエンドヘッダー
    - このカテゴリに分類されるヘッダーは、リクエストやレスポンスの最後の受信者宛に転送されるもの。
  - ホップバイホップヘッダー
    - 一度の転送に対して有効で、キャッシュやプロ棋士によって転送されたりしないものもある。

## 6.3 HTTP/1.1 一般ヘッダーフィールド
- 一般ヘッダーフィールドは、リクエストメッセージとレスポンスメッセージの両方で使用されるヘッダー
- Cache-Control
  - ディレクティブと呼ばれるコマンドを指定することで、キャッシングの動作を指定する
- Connection
  - プロキシに対してそれ以上転送しないヘッダーフィールドの指定
  - 持続的接続の管理
- Date
  - HTTPメッセージを生成した日時を示す。
- Pragma
  - HTTP/1.1より古いバージョンのなごりで、HTTP/1.0との後方互換性のためだけに定義されているヘッダーフィールド
  - Pragma: no-cacheと指定でき、クライアントが全ての中間サーバーに対して、キャッシュされたリソースでの応答を望まないことを要求するために使用される。
- Trailer
  - メッセージボディの後に記述されているヘッダーフィールドをあらかじめ伝えることができる。HTTP/1.1で実装されているチャンク転送エンコーディングを使用している場合に使うことができる。
- Transfer-Encoding
  - メッセージボディの転送コーディング形式を指定する場合に使用される。
  - HTTP/1.1において転送コーディング形式は、チャンク転送コーディングのみが定義されている。
- Upgrade
  - HTTPおよび他のプロトコルの新しいバージョンが通信に利用される場合に使用される。
- Via
  - クライアントとサーバー間でのリクエストまたはレスポンスのメッセージの経路を知るために使用される。
  - Viaヘッダーフィールドは、転送されるメッセージの追跡や、リクエストループの回避などに用いられるため、右プロキシを経由する場合には必ず付加する必要がある。
- Warning
  - レスポンスに関する追加情報を伝える。基本的にはキャッシュに関する問題の警告をユーザーに伝える。

## 6.4 リクエストヘッダーフィールド
- Accept
  - ユーザーエージェントが処理することができるメディアタイプと、メディアタイプの相対的な優先度を伝えるために使用される。
- Accept-Charset
  - ユーザーエージェントが処理することができる文字セットと、文字セットの相対的な優先度を伝えるために使用される。
- Accept-Encoding
  - ユーザーエージェントが処理することができるコンテンツコーディングと、コンテンツコーディングの相対的な優先度を伝えるために使用される。
- Accept-Language
  - ユーザーエージェントが処理することができるし全言語のセット（日本語や英語という意味）と、自然言語セットの相対的な優先度を伝えるために使用される。
- Authorization
  - ユーザーエージェントの認証情報（credentials値）を伝えるために使用される。
- Expect
- From
- Host
  - リクエストしたリソースのインターネットホストとポート番号を伝える。
  - HTTP/1.1で唯一の必須ヘッダーフィールド
  - 同じIPアドレスで複数のドメインが運用されていた場合に、どのドメインに対してのリクエストなのかを識別するために使う。
- If-Match
  - "If-xxx"という書式のリクエストヘッダーフィールドは、条件付きリクエストと呼ばれる。条件付きリクエストを受け取ったサーバーは、指定された条件が真の場合にのみリクエストを受け付ける。
  - If-Matchは、サーバー上のリソースを特定するためにエンティティタグ（ETag）値を伝える。
- If-Modified-Since
  - 指定されたフィールド値の日時以降にリソースが更新されていればリクエストを受け付ける。
- If-None-Match
- If-Range
- If-Unmodified-Since
- Max-Forwards
  - TRACEまたはOPTIONSメソッドによるリクエストの際に転送しても良いサーバー数の最大値を10進数の整数で指定する。
  - サーバーは、次のサーバーにリクエストを転送する際には、Max-Forwardsの値から1引いたものをセットし直す。Max-Forwardsの値が0のリクエストを受け取った場合には、転送せずにレスポンスを返す必要がある。
- Proxy-Authorization
- Range
- Referer
  - リクエストが発生した元のリソースのURIを伝える。
- TE
- User-Agent

## 6.5 レスポンスヘッダーフィールド
- Accept-Ranges
- Age
- ETag
- Location
- Proxy-Authenticate
- Retry-After
- Server
  - サーバーに実装されているHTTPサーバーのソフトウェアを伝える。
- Vary
  - オリジンサーバーからVaryで指定されたレスポンスを受け取ったプロキシサーバーは、以後はキャッシュしたときのリクエストと同様のVaryに指定されているヘッダーフィールドを持つリクエストに対してのみキャッシュを返すことができる。
- WWW-Authenticate

## 6.6 エンティティヘッダーフィールド
- Allow
- Content-Encoding
- Content-Language
- Content-Length
- Content-Location
- Content-MD5
  - コンテンツが偶発的に変更されたことは知ることができるが、悪意を持った改竄は検出できない。
- Content-Range
- Content-Type
  - エンティティボディに含まれるオブジェクトのメディアタイプを伝える。
- Expires
- Last-Modified

## 6.7 Cookieのためのヘッダーフィールド

### Set-Cookie
- サーバーがクライアントに対して状態管理を始める際に、さまざまな情報を伝える。
- フィールド値には次の情報が記される。
  - NAME=VALUE　：　Cookieにつける名前とその値（必須）
  - expires=DATE　：　Cookieの有効期限（指定しない場合はブラウザを閉じるまで）
  - path=PATH　：　Cookieの適用対象となるサーバー上のディレクトリ（指定しない場合はドキュメントと同じディレクトリ）
  - domain=ドメイン名　：　Cookieの適用対象となるドメイン名（指定しない場合はCookieを生成したサーバーのドメイン名）　後方一致になるので、明示的に複数のドメインに対してCookieを送出する場合を除いて、domain属性は指定しない方が安全。
  - Secure　：　HTTPSで通信している場合にのみCookieを送信
  - HttpOnly　：　CookieをJavaScriptからアクセスできないように制限する。クロスサイトスクリプティングからCookieの盗聴を防ぐことを目的としている。

### Cookie
- クライアントがHTTPの状態管理のサポートを望むときに、サーバーから受け取ったCookieを以後のリクエストに含めて伝える。Cookieを複数受け取っている場合には、複数のクッキーを送ることもできる。

## 6.8 その他のヘッダーフィールド
- HTTPヘッダーフィールドは、独自に拡張を行うことができる。よく使うのは以下。
- X-Frame-Options
  - 他のWebサイトのフレームでの表示を制御するHTTPレスポンスヘッダーで、クリックジャッキングという攻撃を防ぐことを目的としている。
- X-XSS-Protection
  - クロスサイトスクリプティング(XSS)対策としてのブラウザのXSS保護機能を制御するHTTPレスポンスヘッダーです。
- DNT
- P3P

# 第7章 Webを安全にするHTTPS

## 7.1 HTTPの弱点
- HTTPは主に次のような弱点を持っている。
  - 通信が平文（暗号化しない）なので盗聴可能
  - 通信相手を確かめないのでなりすまし可能
  - 完全性を証明できないので改竄可能

### 通信が平文なので盗聴可能
- 通信が平文であることが弱点になる理由は、TCP/IPの仕組みとして通信の内容はすべて通信経路の途中で覗き見ることができるからである。暗号化されている場合も同様に通信内容を除き見れるが、メッセージの中身の意味は読み取れない。
- 盗聴から情報を守るための方法で最も普及している技術が暗号化。暗号化にはいくつかの対象がある。
  - 通信の暗号化
    - SSLやTLSというHTTPとは別のプロトコルを組み合わせることで、HTTPの通信内容を暗号化できる。
    - SSLなどによって安全な通信路を確立してから、その通信路を使ってHTTPの通信を行う。このSSLを組み合わせたHTTPは、HTTPSやHTTP over SSLと呼ばれる。
  - コンテンツの暗号化
    - もう1つは、通信しているコンテンツの内容自体を暗号化してしまうこと。
    - この場合、クライアント側でHTTPメッセージを暗号化して出力するような処理が必要になる。
    - 通信経路を暗号化するわけではないので、改竄やなりすましの危険性などが残る。

### 通信相手を確かめないのでなりすまし可能
- HTTPによる通信は、相手が誰かを確かめる処理はない。それゆえにシンプルな実装になり普及したという面もあるが、弱点に繋がることもある。
  - リクエストを送った先のWebサーバーが、なりすましたWebサーバーである恐れがある。
  - 通信相手が、アクセスを許可された者であるかどうか確認できない。
  - どこの誰がリクエストしたのかを確認できない。
  - 意味のないリクエストでも受け付けてしまうので、大量のリクエストによるDoS攻撃を防ぐことができない。
- SSLは暗号化だけでなく、証明書と呼ばれる相手を確かめる手段を提供している。
  - 通信相手のサーバーやクライアントが持っている証明書を確認することで、通信相手が本来意図した相手であるかどうかを判断することができる。

### 完全性を証明できないので改竄可能
- 完全性というのは情報の正確さを指す。HTTPが完全性を証明できないということは、もしリクエストやレスポンスが発信されてから相手が受け取るまでの間に改竄されたとしても、それを知ることができないということである。
- 途中で攻撃者によってリクエストやレスポンスを横取りされて改竄される攻撃を中間者攻撃と呼ぶ。
- HTTPを使って完全性を確かめるための方法はあるが、確実で便利な方法は今のところ存在しない。その中でもよく使われている方法は、MD5やSHA-1などのハッシュ値を確かめる方法と、ファイルのデジタル署名を確認する方法である。
- 改竄を確実に防ぐにはHTTPSを使う必要がある。SSLには認証や暗号化、そしてダイジェスト機能を提供している。

## 7.2 HTTP+暗号化+認証+完全性保護=HTTPS
- 暗号化や認証などの仕組みをHTTPに付け加えたものをHTTPS（HTTP Secure）と呼ぶ。
- HTTPSは新しいアプリケーション層のプロトコルではない。HTTPの通信を行うソケット部分をSSLやTLSというプロトコルに置き換えているだけ。SSLを使用した場合には、HTTPはSSLと通信し、SSLがTCPと通信するようになる。
- SSLを使うことで、HTTPはHTTPSとして暗号化と証明書と完全性保護を利用することができるようになる。
- SSLはHTTPだけでなくアプリケーション層で動作する他のプロトコルにも使うことができる。SSLは現在最も広く使われているネットワークセキュリティ技術と言える。
- SSLでは公開鍵暗号方式と呼ばれる暗号化方式を採用している。現代の暗号は、アルゴリズムは公開されていて鍵を秘密にすることで安全性を保つ。暗号化や復号には、この鍵を使う。

#### 共通鍵暗号
- 暗号化と復号に同じ1つの鍵を使用する方式を共通鍵暗号と呼ぶ。
- 共通鍵暗号方式は、相手に鍵を渡さなければならないが、相手に安全に鍵を渡すのが難しいのが問題である。

#### 公開鍵暗号
- 公開鍵暗号では異なる2つの鍵ペアを使う。片方は秘密鍵、もう一方は公開鍵と呼ばれる。
- 公開鍵暗号を使った暗号化は、暗号を送る側が相手の公開鍵を使って暗号化を行う。そして、暗号化された情報を受け取った相手は自分の秘密鍵を使って復号を行う。この方式は、暗号を解く秘密鍵を通信で送る必要がないので、鍵を盗聴で盗まれる心配がない。

#### HTTPSはハイブリッド暗号システム
- HTTPSは、共通鍵暗号と公開鍵暗号の両方を使うハイブリッド暗号システムである。鍵を安全に交換できるなら、公開鍵暗号だけを使って通信しても良いと考えるかもしれないが、公開鍵暗号は共通鍵暗号に比べて処理速度が遅い。
- そこで、それぞれの方式を組み合わせて通信を行う。具体的には、共通鍵暗号で使う鍵を交換するところまでは公開鍵暗号を使い、その後の通信でメッセージを交換するところでは共通鍵暗号を使う。

### 公開鍵が正しいかどうかを証明する証明書
- 公開鍵暗号方式の問題点は、公開鍵が本物であるかどうかを証明できないということ。
- この問題の解決には、認証局（CA：Certificate Authority）とその期間が発行する公開鍵証明書が利用される。
- 認証局とは、クライアントとサーバーが双方ともに信頼する第三者機関がその立場に立つ。認証局は証明機関と呼ばれることもある。
- 認証局の利用方法は下記
  1. サーバーの運営者が、サーバーの公開鍵を認証局に登録
  2. 認証局は、認証局の秘密鍵でサーバーの公開鍵にデジタル署名して公開鍵証明書を作成し、サーバーに送る
  3. サーバーは公開鍵証明書をクライアントに送って、公開鍵暗号で通信を開始する。
  4. クライアントは、サーバーの公開鍵証明書（サーバーの公開鍵+認証局のデジタル署名からなる。デジタル証明書とも呼ぶ。）を入手し、デジタル署名を認証局の公開鍵で検証し、公開鍵が本物かを確認
    - 認証局の公開鍵はあらかじめブラウザに組み込まれている。
  5. クライアントは、サーバーの公開鍵で暗号化しメッセージを送信
  6. サーバーは、サーバーの秘密鍵でメッセージを復号

#### 組織の実在性を証明するEV SSL証明書
- 証明書には、相手が実在する企業であるかを確かめるという役割もある。そういう役割を持った証明書をEV SSL証明書と言う。
- EV SSL証明書で証明されたWebサイトは、ブラウザのアドレスバーの色が緑色になるので視覚的にわかる。これはフィッシング詐欺の防止を意図したものだが、多くのユーザーはEV SSL証明書のことを知らないので、効果のほどは疑問である。

#### クライアント証明書
- HTTPSではクライアント証明書も利用できる。これを利用することで、サーバー証明書と同じく、サーバーが通信している相手が意図したクライアントであることを証明するクライアント認証を行うことができる。
- クライアント証明書にはいくつかの問題点がある。
  - 証明書の入手と配布の問題　ユーザーが有料で購入する必要がある
  - クライアント証明書はあくまでクライアントの実在を証明するだけで、ユーザーの実在を証明するわけではない
    - クライアント証明書が入ったコンピュータを使う権限があれば、クライアント証明書を利用できてしまう。

#### 自己認証局発行の証明書はオレオレ証明書
- OpenSSLなどのソフトウェアを使えば、誰でも認証局を構築することができ、サーバー証明書を発行できる。しかし、このサーバー証明書はインターネット上では証明書としての用をなさない。
- 独自に構築した認証局を自己認証局と呼び、そこが発行した役に立たない証明書を揶揄してオレオレ証明書と呼ぶことがある。
- オレオレ証明書を使ったWebサイトにブラウザでアクセスすると警告メッセージが表示される。
- オレオレ証明書が役に立たないのは、なりすましの可能性を払拭できないからである。

### 安全な通信を行うHTTPSの仕組み
- 通信手順は12段階ほどある。詳細は省略

#### SSLとTLS
- SSL3.0をベースにしたTLS1.0が作成され、TLS1.1、TLS1.2がある。現在はSSL3.0、TLS1.0が主流である。
- TLSはSSLを元にしたプロトコルだが、これらを総称してSSLと呼ぶこともある。

#### SSLは遅い
- HTTPSの問題点として、SSLを使うと処理が遅くなると言うことがある。
- SSLの遅さには2種類ある。
  - 通信が遅くなる TCP接続とHTTPのリクエスト・レスポンス以外にSSLに必要な通信が入るため
  - CPUやメモリなどのリソースを多量に消費することで処理が遅くなる 暗号化や複合のための計算にリソースを割く。

# 第8章 誰がアクセスしているかを確かめる認証

## 8.1 認証とは
- システムにアクセスする権利を持った本人であるかどうかを確かめるために、「登録した本人しか知らない情報」や「登録した本人しか持っていない情報」などで確認する。
  - その情報として、パスワード、ワンタイムトークン、電子証明書、バイオメトリクス、ICカードなどを使う。
- HTTP/1.1で利用できる認証方式には次のものがある。
  - BASIC認証
  - DIGEST認証
  - SSLクライアント認証
  - フォームベース認証

## 8.2 BASIC認証
- HTTP/1.0から実装されている認証方式。Webサーバーと対応しているクライアントの間で行われる認証方式。

### 認証手順
1. リクエスト送信
2. 認証が必要であることを伝えるステータスコード401で応答
3. ユーザーIDとパスワードをBase64形式でエンコードしたものを送信
4. 認証成功時にはステータスコード200で応答し、失敗した場合には再度ステータスコード401で応答

- BASIC認証ではBase64と言うエンコーディング形式を使っているが、これは絵暗号化ではないので、暗号化されていない通信経路上でBASIC認証を行い、盗聴された場合にはユーザーIDとパスワードが盗まれてしまう可能性がある。
- BASIC認証は、使い勝手の問題や、多くのWebサイトで求められるセキュリティレベルには足りていないことから、それほど使われていない。

## 8.4 DIGEST認証
- BASIC認証の弱点を補うものとしてHTTP/1.1から実装された**DIGEST認証**がある。
- これには**チャレンジレスポンス方式**が使われていて、BASIC認証のように生のパスワードを直接送ることはない。
- チャレンジレスポンス方式は、最初にクライアントがサーバーに対して認証要求を送り、サーバー側から受け取った**チャレンジコード**を使って、**レスポンスコード**を計算する。その値をサーバーに送信して認証を行う方法である。

### 認証手順
1. リクエスト送信
2. 認証が必要であることを伝えるステータスコード401で応答するとともにチャレンジコード（nonce）を送信
3. パスワードとチャレンジコードからレスポンスコード（response）を計算して送信
4. 認証成功時にはステータスコード200で応答し、失敗した場合には再度ステータスコード401で応答

- DIGEST認証ではパスワードの盗聴を防ぐための保護機能は提供するが、それ以外になりすましを防ぐといった機能は提供していない。
- DIGEST認証もまたBASIC認証と同様に、使い勝手の問題や、多くのWebサイトで求められるセキュリティレベルには足りていないことから、それほど使われていない。

## 8.4 SSLクライアント認証
- ユーザーIDとパスワードを使った認証方式ではなりすましされるリスクがある。それを防ぐための対策の1つとして、**SSLクライアント認証**が使われることがある。
- SSLクライアント認証は、HTTPSの**クライアント証明書**を利用した認証方式。
- SSLクライアント認証は、多くの場合後述するフォームベース認証と併せた**二要素認証**の1つとして利用される。
  - 二要素認証とは、例えばパスワードという1つの要素だけではなく、利用者が持つ別の情報を併用して認証を行う方法。
  - つまり、1つ目の認証情報としてSSLクライアント認証を使ってクライアントのコンピューターを認証し、もう1つの認証情報としてパスワードを使うことで、ユーザーの本人確認を行う。
- SSLクライアント認証では、クライアント証明書を利用する必要があり、利用する他にはコストが必要になる。

## 8.5 フォームベース認証
- フォームベース認証はHTTPのプロトコルとして仕様が定義されている認証方式ではない。サーバー上のWebアプリケーションに、クライアントが**資格情報**（Credential）を送信し、その資格情報の検証結果によって認証を行う方式。
- 多くの場合、資格情報としてあらかじめ登録してあるユーザーIDとパスワードをユーザーに入力させる。

### 認証の大半はフォームベース認証
- Webサイトの認証機能として求める機能のレベルを満たした標準的なものが存在しないため、Webアプリケーションで各々が実装するフォームベース認証を採用するしかない。

### セッション管理とCookieによる実装
- フォームベース認証は、標準的な仕様が決められていないが、一般的によく使われている方法としては、**セッション管理**のために**Cookie**を使用する実装方法がある。これらを利用して、HTTPにはない状態管理の機能を補う。
- フォームベース認証では、パスワードなどの資格情報をサーバー側にどのように保存するべきかということも標準化されていない。一般的には、安全な方法としてパスワードを**salt**という負荷情報を使って**ハッシュ**というアルゴリズムで計算した値を保持する。

#### 手順
1. クライアント側がユーザーIDやパスワードなどの資格情報を含めたリクエストをHTTPS通信でサーバーに送信する。
2. サーバー側は、ユーザーを識別するためのセッションIDを発行する。クライアントから受信した資格情報を検証することで認証を行い、そのユーザーの認証状態をセッションIDと紐付けてサーバー側に記録する。
3. サーバー側からセッションIDを受け取ったクライアントは、Cookieとして保存しておく。次にサーバーにリクエストを送信する際には、ブラウザは自動的にCookieを早出するので、セッションIDがサーバーにが送信される。サーバー側では受信したセッションIDを検証することで、ユーザーや認証状態を識別できる。

# 第9章 HTTPに機能を追加するプロトコル

## 9.1 HTTPをベースにしたプロトコル
- HTTPをベースとして、それに追加する形で新たなプロトコルがいくつも実装されてきた。

## 9.2 HTTPのボトルネックを解消するSPDY
- 現状のHTTPでは、いくつかの仕様がボトルネックになり、サーバー上で更新された情報をリアルタイムにクライアントに反映することが難しい。
- AjaxやCometなどの技術である程度の改善は行われているが、HTTPのプロトコルの制約は残っている。SPDYはHTTPが抱えているボトルネックをプロトコルレベルで解消するために、開発が進められているプロトコルである。

## 9.3 ブラウザで双方向通信を行うWebSocket
- **WebSocket**は、WebブラウザとWebサーバーのための双方向通信の規格。主にAjaxやCometで使うXMLHttpRequestの欠点を解決するための技術として開発が進められている。

### WebSocketプロトコル
- WebSocketは、Webサーバーとクライアントが一度接続を確立したあと、その後の通信をすべて専用プロトコルで行う。接続の起点がクライアントからであることはHTTPと変わらないが、一度接続を確立すると、サーバーとクライアントのどちらからでも送信を行うことが可能。
- 主な特徴
  - サーバープッシュ機能
  - 通信量の削減
- WebSocketで通信を行うには、一度HTTPで接続を確立後、ハンドシェイクというリクエスト・レスポンスの手続きを行う必要がある。ハンドシェイクによってWebSocketコネクションが確立した後は、HTTPではなく、WebSocket独自のデータフレームを用いて通信を行う。

#### WebSocket API
- JSからWebSocketプロトコルを使った双方向通信を行うためには、W3Cで仕様が策定されている「The WebSocket API(http:// www.w3.org/TR/websockets/)」で提供されているWebSocketインタフェースを使う。

## 9.4 登場が待たれるHTTP/2.0

## 9.5 Webサーバー上のファイル管理を行うWebDAV

# 第10章 Webコンテンツで使う技術

## 10.1 HTML

## 10.2 ダイナミックHTML

## 10.3 Webアプリケーション
- 動的コンテンツ
- CGI
- サーブレット

## 10.4 データ配信に利用されるフォーマットや言語
- XML
- RSS, Atom
- JSON - RFC4627「The application/json Media Type for JavaScript Object Notation(JSON)」を参照

# 第11章 Webへの攻撃技術

## 11.1 Webへの攻撃技術
- 攻撃の対象となるのは、HTTPを実装したサーバーやクライアント、そしてサーバー上で動作するWebアプリケーションなどのリソース。現在、インターネットからの攻撃の大半がWebサイトを狙ったものであり、その中でも特にWebアプリケーションを対象にした攻撃が数多く行われている。本性では主にWebアプリケーションへの攻撃について説明を行う。
- 世間のWebアプリは、攻撃者が悪用することができる脆弱性というバグを抱えた状態のまま稼働しているものがある。
- リクエストはクライアント側で改竄可能。Webアプリへの攻撃は、HTTPリクエストメッセージに攻撃コードを載せて行われる。
- Webアプリへの攻撃パターンは以下の2つがある。
  - 能動的攻撃
    - 攻撃者がWebアプリにアクセスし、攻撃コードを送るタイプの攻撃
    - 代表的な攻撃：　SQLインジェクション、OSコマンドインジェクション
  - 受動的攻撃
    - 罠を利用してユーザーに攻撃コードを実行させる攻撃。
      1. 罠へ誘導（Webページやメールに罠を仕掛ける）
      2. 仕掛けられたHTTPリクエストをユーザーのブラウザなどで開く
      3. ユーザーのブラウザが攻撃コードを実行
      4. 攻撃コードを実行した結果、ユーザーが持つCookieなどの奪取や権限の悪用が行われる
    - 代表的な攻撃：　クロスサイト・スクリプティング、クロスサイト・リクエストフォージェリ
    - 受動的攻撃を利用すると、イントラネットなどのインターネットから直接アクセスすることができないネットワークに対しての攻撃を行うことができる。

## 11.2 出力値のエスケープの不備による脆弱性
- Webアプリのセキュリティ対策を実施する箇所は大きく分けると下記
  - クライアント側でのチェック
  - Webアプリケーション側（サーバー側）でのチェック
    - 入力値のチェック
    - 出力値のエスケープ
- Webアプリで扱ったデータを、DBやファイルシステムなどに出力する際に、その出力先に応じた値のエスケープがセキュリティ対策としては重要になる。

### クロスサイト・スクリプティング（Cross-Site Scripting: XSS）
- クロスサイト・スクリプティングは、脆弱性のあるWebサイトのユーザーのブラウザ上で、不正なHTMLタグやJavaScriptなどを動かす攻撃。動的にHTMLを生成する箇所で脆弱性が発生する可能性がある。これは、攻撃者が作成したスクリプトを罠として、ユーザーのブラウザ上で動かす受動的攻撃である。
- XSSによって、次のような影響を受ける可能性がある。
  - 偽の入力フォームなどによってユーザーの個人情報が盗まれる
  - スクリプトによってユーザーのCookieの値が盗まれたり、被害者が意図しないリクエストの送信が行われる
  - 偽の文章や画像などが表示される

### SQLインジェクション
- Webアプリケーションが利用しているデータベースに対して、SQLを不正に実行する攻撃。大きな脅威を引き起こす可能性がある脆弱性で、個人情報や機密情報漏洩に直結することもある。
- SQLインジェクションによって、次のような影響を受ける可能性がある。
  - データベースないのデータの不正な閲覧や改竄
  - 認証の回避
  - データベースサーバーを経由したプログラムの実行など
- SQLインジェクションは、攻撃者によって開発者が意図しない形にSQL文が改変され、構造が破壊される攻撃である。

### OSコマンドインジェクション
- Webアプリを経由して、OSコマンドを不正に実行する攻撃。シェルを呼び出す関数があるところで発生する可能性がある。

### HTTPヘッダーインジェクション
- レスポンスヘッダーフィールドに攻撃者が改行コードなどを挿入することで、 任意のレスポンスヘッダーフィールドやボディを追加する攻撃で、受動的攻撃である。
- 特にボディを追加する攻撃はHTTPレスポンス分割攻撃と呼ばれる。
- 次のような影響を受ける可能性がある。
  - 任意のCookieのセット
  - 任意のURLへのリダイレクト
  - 任意のボディの表示（HTTPレスポンス分割攻撃）

### メールヘッダーインジェクション
- Webアプリのメール送信機能において、攻撃者が任意のToやSubjectなどのメールヘッダーを不正に追加する攻撃。脆弱性のあるWebサイトを利用して、迷惑メールやウイルスメールなどを任意の宛先に送信することが可能になる。

### ディレクトリ・トラバーサル
- 公開することを意図していないディレクトリのファイルに対して、不正にディレクトリパスを横断してアクセスする攻撃。この攻撃は、パストラバーサル(Path Traversal)と呼ぶこともある。

### リモート・ファイル・インクルージョン
- スクリプトの一部を別ファイルから読み込む際に、攻撃者が指定した外部サーバーのURLをファイルとして読み込ませることで、任意のスクリプトを動作させる攻撃。
- 主にPHPに存在する脆弱性。PHPのある機能を使うとできるが、危険なのでPHP5.2.0以降ではデフォルトで無効になっている。

## 11.3 設定や設計の不備による脆弱性

### 強制ブラウジング
- 強制ブラウジング(Forced Browsing)は、Webサーバーの公開ディレクトリに配置されているファイルのうち、公開することを意図していないファイルが閲覧可能になっている脆弱性。

### 不適切なエラーメッセージ処理
- 攻撃者にとって有益な情報がWebアプリケーションのエラーメッセージに含まれるという脆弱性。

#### Webアプリケーションによるエラーメッセージ
- 例えば、認証画面で「メールアドレスは登録されていません」とエラーメッセージを出すと、アカウントの存在確認に利用される可能性がある。エラーメッセージを攻撃のヒントとして利用されないようにするために、「認証エラーです」程度の内容に留めておく。

#### DBなどのシステムによるエラーメッセージ
- エラーメッセージを攻撃のヒントとして利用されないようにするために、各システムの設定により詳細なエラーメッセージを抑制するか、カスタムエラーメッセージを利用する。

### オープンリダイレクト
- 指定した任意のURLにリダイレクトする機能である。リダイレクト先のURLに悪意のあるWebサイトが指定された場合、ユーザーがそのWe サイトに誘導されてしまう脆弱性につながる。
- ユーザーが信頼しているWebサイトにオープンリダイレクトの機能がある場合、攻撃者はそれを利用してフィッシング詐欺などを仕掛ける可能性がある。

## 11.4 セッション管理の不備による脆弱性

### セッションハイジャック
- 攻撃者がユーザーのセッションIDを何らかの方法で入手し、そのセッションIDを不正に利用することで、ユーザーになりすます攻撃。
- 攻撃者がセッションIDを入手する方法は、主に下記。
  - 不適切な生成方法によるセッションIDの推測
  - 盗聴やXSSなどによるセッションIDの盗用
  - セッション固定攻撃によるセッションIDの強制

### セッションフィクセーション
- 攻撃者が指定したセッションIDをユーザーに強制的に使わせるという攻撃で、受動的攻撃。セッション固定攻撃とも呼ばれる。

### クロスサイト・リクエストフォージェリ(CSRF)
- 認証済みのユーザーが意図しない個人情報や設定情報など何らかの状態を更新するような処理を、攻撃者が仕掛けた罠によって強制的に実行させるという攻撃で、受動的攻撃である。
- この攻撃により次のような影響を受ける可能性がある
  - 認証済みのユーザーの権限で設定情報などの更新
  - 認証済みのユーザーの権限で商品を購入
  - 認証済みのユーザーの権限で掲示板への書き込み

## 11.5 その他

### パスワード・クラッキング
- パスワードを割り出して認証を突破する攻撃。
- 下記の方法などがある。
  - ネットワーク経由でのパスワード試行
  - 暗号化されたパスワードの解読(攻撃者がシステムに侵入するなどして、暗号化やハッシュ化されたパスワードのデータを取得済みという状況)

#### ネットワーク経由でのパスワード試行
- 総当たり攻撃
- 辞書攻撃

#### 暗号化されたパスワードの解読
- 通常Webアプリでパスワードを保存する場合は、ハッシュ化やsaltなどの方法で暗号化されている。攻撃者がこれを手に入れたとしても、平文を手に入れる必要がある。
- 暗号化されたデータから平文を導き出す方法は以下がある。
  - 総当たり攻撃・辞書攻撃による類推
  - レインボーテーブル
  - 鍵の入手
  - 暗号アルゴリズムの脆弱性

### クリックジャッキング
- 透明なボタンやリンクを罠となるWebページに埋め込み、そのリンクをユーザーにクリックさせることで意図しないコンテンツにアクセスさせる攻撃。UI Redressingと呼ばれることもある。

### DoS攻撃
- サービスの提供を停止状態にする攻撃。サービス停止攻撃やサービス拒否攻撃と呼ばれることもある。
- 主なDoS攻撃には以下の方法がある。
  - アクセスを集中させることで負荷を掛けてリソースを使い切らせることで事実上サービスを停止状態にする
  - 脆弱性を攻撃することでサービスを停止させる

### バックドア
- 制限された機能を正規の手続きを踏まずに利用するために設けられた裏口。バックドアを使うことで、本来の制限を超えた機能を利用することができる。
- バックドアには以下の種類がある。
  - 開発段階のデバッグ用に組み込んだバックドア
  - 開発者が自己利益のために組み込んだバックドア
  - 攻撃者が何らかの方法で設置したバックドア
