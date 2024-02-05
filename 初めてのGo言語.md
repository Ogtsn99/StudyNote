# Go環境のセットアップ
macならこれでよし
```
brew install go
```
linuxなら
```
$ tar -C /usr/local -xzf go1.19.linux-arm64.tar.gz
$ echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
$ source $HOME/.profile
```
サードパーティのモジュールやパッケージはホームディレクトリの下のgoディレクトリの下にダウンロードされる。コマンドはgo/binディレクトリの下。

go run hello.go は以下のような動作になっている
1. バイナリファイルがビルドされ、一時ディレクトリに置かれる
2. そのファイルを実行する
3. プログラム終了後、そのファイルが削除される
再利用できるように実行形式のバイナリファイルを作るには
```
go build hello.go
```
-oで、名前をつけることが可能
```
go build -o hello_world hello.go
```
go installはパッケージをダウンロードし、go/binに置く。

go fmtは便利なフォーマットツールであるが、その機能強化版であるgoimportsというツールもある。これは、フォーマットだけでなく、import文をクリーンにするのに使える

lintを使うとコードのスタイルがガイドラインに沿っているかを確認できる。一つのツールはstaticcheckである。
go install honnef.co/go/tools/cmd/staticcheck@latest でインストール可能
```
staticcheck
```
golangci-lintならいろんなスタイル関係のチェック（staticcheck, go vetなど）を一括でしてくれる。
```
golangci-lint run
```
.golangcli.ymlで、どのリンターを実行し、どのファイルを分析するかを指定可能。
詳細は https://golangci-lint.run/usage/configuration/

# 2章 基本型と宣言