---
name: Beeツールの使用方法
sort: 2
---

# beeツールの紹介

Beeツールは急速にbeegoを開発できるプロジェクトです。beeツールで自動コンパイルを作成し、リロード、開発、テスト、簡単にbeegoアプリケーションをデプロイすることができます。

## beeツールのインストール

次のコマンドを使用してbeeツールをインストールすることができます:

	go get github.com/beego/bee

`bee`はデフォルトで`GOPATH/bin`にインストールされます。`PATH`に`GOPATH/bin`を追記する必要があり、さもなくば`bee`コマンドが動作しません。

## Beeツールコマンド

コマンドラインで`bee`をタイプしてください。以下のメッセージが表示されると思います:

```
Bee is a tool for managing beego framework.

Usage:

	bee command [arguments]

The commands are:

	new         Create a Beego application
	run         run the app and start a Web server for development
	pack        Compress a beego project into a single file
	api         create an API beego application
	bale        packs non-Go files to Go source files
	version     show the Bee, Beego and Go version
	generate    source code generator
	migrate     run database migrations
```

### `new`コマンド

`new`コマンドは新たなWebプロジェクトを生成します。新たなbeegoプロジェクトを`$GOPATH/src`直下で、`bee new <project name>`とタイプすることで生成します。これらは全てのプロジェクトのフォルダとファイルです:

```
bee new myproject
[INFO] Creating application...
/gopath/src/myproject/
/gopath/src/myproject/conf/
/gopath/src/myproject/controllers/
/gopath/src/myproject/models/
/gopath/src/myproject/routers/
/gopath/src/myproject/tests/
/gopath/src/myproject/static/
/gopath/src/myproject/static/js/
/gopath/src/myproject/static/css/
/gopath/src/myproject/static/img/
/gopath/src/myproject/views/
/gopath/src/myproject/conf/app.conf
/gopath/src/myproject/controllers/default.go
/gopath/src/myproject/views/index.tpl
/gopath/src/myproject/routers/router.go
/gopath/src/myproject/tests/default_test.go
/gopath/src/myproject/main.go
13-11-25 09:50:39 [SUCC] New application successfully created!
```

```
myproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── models
├── routers
│   └── router.go
├── tests
│   └── default_test.go
├── static
│   ├── js
│   ├── css
│   └── img
├── views
│   └── index.tpl
├── main.go

11 directories, 5 files
```

### Command `run`

コンパイルし、それらを毎回手動で実行する必要があるという問題を抱えています。 `bee run`コマンドはbeegoプロジェクトフォルダの範囲内で、すべての変更後、すぐに結果を見ることができるよう[inotify](http://en.wikipedia.org/wiki/Inotify) を使用して、任意のbeegoプロジェクトのファイルシステムを監視します。

```
cd myproject
bee run
13-11-25 09:53:04 [INFO] Uses 'myproject' as 'appname'
13-11-25 09:53:04 [INFO] Initializing watcher...
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/controllers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/routers)
13-11-25 09:53:04 [TRAC] Directory(/gopath/src/myproject/tests)
13-11-25 09:53:04 [INFO] Start building...
13-11-25 09:53:16 [SUCC] Build was successful
13-11-25 09:53:16 [INFO] Restarting myproject ...
13-11-25 09:53:16 [INFO] ./myproject is running...
13-11-25 09:53:16 [app.go:103] [I] http server Running on :8080
```
Upon execution of this command, open your browser and visit `http://localhost:8080/`, you should see your app running:
上のコマンドを実行すると、Webブラウザを開き、`http://localhost:8080/` に行ってください、アプリケーションの実行が表示されます:

![](../images/beerun.png)

After modifying the `default.go` file in the `controllers` folder, we can see the output below from the command line:
` controllers`フォルダ内の` default.go`ファイルを変更した後、我々は、コマンドラインから次の出力を見ることができます：

```
13-11-25 10:11:20 [EVEN] "/gopath/src/myproject/controllers/default.go": DELETE|MODIFY
13-11-25 10:11:20 [INFO] Start building...
13-11-25 10:11:20 [SKIP] "/gopath/src/myproject/controllers/default.go": CREATE
13-11-25 10:11:23 [SKIP] "/gopath/src/myproject/controllers/default.go": MODIFY
13-11-25 10:11:23 [SUCC] Build was successful
13-11-25 10:11:23 [INFO] Restarting myproject ...
13-11-25 10:11:23 [INFO] ./myproject is running...
```

Refreshing the browser should show the results of the new modifications.
ブラウザをリフレッシュすると、新しい変更結果が表示されるはずです。

### Command pack

` pack`コマンドは、プロジェクトを単一のファイルへ圧縮するために使用されます。そうすれば、アップロードしてサーバ上にzipファイルを解凍することで、プロジェクトを展開することができます。

```
bee pack
app path: /gopath/src/apiproject
GOOS darwin GOARCH amd64
build apiproject
build success
exclude prefix:
exclude suffix: .go:.DS_Store:.tmp
file write to `/gopath/src/apiproject/apiproject.tar.gz`
```

プロジェクトフォルダ内に圧縮ファイルが見える問思います:

```
rwxr-xr-x  1 astaxie  staff  8995376 11 25 22:46 apiproject
-rw-r--r--  1 astaxie  staff  2240288 11 25 22:58 apiproject.tar.gz
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 conf
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 controllers
-rw-r--r--  1 astaxie  staff      509 11 25 22:31 main.go
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 models
drwxr-xr-x  3 astaxie  staff      102 11 25 22:31 tests
```

### Command api

The `new` command is used for crafting new web applications. But there are many users who use beego for developing APIs. We can use the `api` command to create new API applications.
Here is the result of running `bee api project_name`:
` new`コマンドは、新しいWebアプリケーションを作り上げるために使用されます。しかし、 APIを開発するためのビーゴを使用する多くのユーザーが存在します。新しいAPIのアプリケーションを作成するために ` api`コマンドを使用することができます。
ここで`ハチのAPI project_name`を実行した結果は次のとおりです:

```
bee api apiproject
create app folder: /gopath/src/apiproject
create conf: /gopath/src/apiproject/conf
create controllers: /gopath/src/apiproject/controllers
create models: /gopath/src/apiproject/models
create tests: /gopath/src/apiproject/tests
create conf app.conf: /gopath/src/apiproject/conf/app.conf
create controllers default.go: /gopath/src/apiproject/controllers/default.go
create tests default.go: /gopath/src/apiproject/tests/default_test.go
create models object.go: /gopath/src/apiproject/models/object.go
create main.go: /gopath/src/apiproject/main.go
```

以下は、新しいAPIアプリケーションの生成されたプロジェクト構造を示しています:

```
apiproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
│   └── object.go
└── tests
    └── default_test.go
```

先ほどの`bee new myproject`コマンドと比較してください。
新しいAPIアプリケーションは、`static`および`views`フォルダが含まれていないことにご注意ください。

### `bale`コマンド

This command is currently only available to the developer team. It's mainly used for compressing all the static files in to a single binary file. So we don't need to carry  static files including js, css, images and views when publish the project. Those files will be self-extracting with non-overwrite when the program starts.
このコマンドは、現在開発チームのみ使用可能です。これは主に、単一のバイナリファイルへのすべての静的ファイルを圧縮するために使用されます。よって、プロジェクトをパブリッシュするのにjs、css、画像やビューを含む静的ファイルを実行する必要はありません。これらのファイルは、プログラムの起動時に上書きしない自己解凍型になります。

### `varsion`コマンド
このコマンドは、`bee` 、`beego`、` go`のバージョンを表示します。

### `create`コマンド
このコマンドは、コントローラ内での関数を解析することにより、ルータを生成します。

### `migrate`コマンド
このコマンドは、データベースをマイグレーションするスクリプトを実行します。

## Beeツールの設定

beeツールのソースコードフォルダに`bee.json`という名前のファイルが存在に築くと思いますが、このファイルはbeegoの設定ファイルです。すべての機能はまだ完了していませんが、今のところ使用したいであろういくつかのオプションがあります:

- `"version": 0`: ファァイルのバージョン、互換性のないフォーマットのバージョンをチェックします。
- `"go_install": false`: `github.com/user/repo/subpkg`のような完全なインポートパスを使用している場合、`go install`を実行することでこのオプションを有効にすることができ、ビルドプロセスをスピードアップすることができます。
- `"watch_ext": []`: 他のファイル拡張子のウォッチを追加します（デフォルトでは`.go`ファイルのみをウォッチ）。例えば、`.ini`、`.conf`などです。
- `"dir_structure":{}`: フォルダ名が MVCの古典的な名前と同じでない場合、変更するためにこのオプションを使用することができます。
- `"cmd_args": []`: 開始ごとのコマンドの引数を追加します。
- `"envs": []`: すべてのための環境変数の設定を開始します。
