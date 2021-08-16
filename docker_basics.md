docker基礎からのコンテナ構築

# 第3章 5分でWebサーバーを起動する
- この章の操作(DockerでApacheを起動する)のまとめ
1. Docker Hubでイメージを探す
2. `docker run`で実行する
3. 停止と再開
  - `docker stop <CONTAINER ID or NAME>`で停止、`docker start`で再開。
  - コンテナの状態は`docker ps`で確認できる（起動中のもののみ）停止中のものも含めて確認するときは、`docker ps -a`とする。
4. ログの確認
  - `docker logs`でログの確認が可能
5. 破棄
  - コンテナを停止しても、コンテナは破棄されず、ディスクスペースを消費する。
  - 完全に削除するには、`docker rm`コマンドを使う。
6. イメージの破棄
  - コンテナを破棄しても、その元となったDockerイメージは残ったまま。
  - 保有しているDockerイメージは`docker image ls`コマンドで確認できる。
  - 使わなくなったイメージは、`docker image rm`コマンドを使って削除する。
