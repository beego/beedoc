# はじめに

## インストール

beegoは、学習やbeegoアプリフレームワークを使用するのに役立つサンプルのアプリケーションが含まれています。

これが動作するために機能するGo 1.1のインストールが必要になります。

beegoと[Bee](http://beego.me/docs/install/bee.md)の開発ツールをインストールする必要があります:
	
	$ go get github.com/astaxie/beego
	$ go get github.com/beego/bee


便宜上、環境変数`$PATH`に`$GOPATH/bin`を追加する必要があります。

それがどのように動作するかすぐに確認したいですか？それでは、単にこのように設定してください:

	$ cd $GOPATH/src
	$ bee new hello
	$ cd hello
	$ bee run

Windowsユーザ：

    > cd %GOPATH%/src
    > bee new hello
    > cd hello
    > bee run hello

These commands help you:
これらのコマンドはあなたを助けます：

1. `$GOPATH`へbeegoをインストールします。
2. お使いのコンピュータへbeeツールをインストールします。
3. `hello`と呼ばれる新しいアプリケーションを作成します。
4. ホットコンパイルを開始します。


一度実行すれば、Webブラウザで[http://localhost:8080/](http://localhost:8080/)を開いてください。

## Simple example

次の例では、Webブラウザに`Hello world`をプリントしていますが、beegoでWebアプリケーションを構築するのがいかに簡単かを示しています。

	package main
	
	import (
		"github.com/astaxie/beego"
	)
	
	type MainController struct {
		beego.Controller
	}
	
	func (this *MainController) Get() {
		this.Ctx.WriteString("hello world")
	}
	
	func main() {
		beego.Router("/", &MainController{})
		beego.Run()
	}

Save file as `hello.go`, build and run it:

	$ go build -o hello hello.go
	$ ./hello

[http://127.0.0.1:8080](http://127.0.0.1:8080)をWebブラウザ上で開いてください、`hello world`が表示されていると思います。

上の例の場面では何が起こったのでしょうか？

1. `github.com/astaxie/beego`パッケージをインポートします。すでにご存知のように、Goはパッケージを初期化し、すべてのパッケージで`init()`を実行します、（[詳細](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/eBook/02.3.md#main-function-and-init-function))、よってbeegoは、この時点で`BeeApp`アプリケーションを初期化します。
2. コントローラを定義してください。匿名のフィールド`beego.Controller`によって`MainController`と呼ばれる構造体を定義していますので、`MainController`は`beego.Controller`が持つすべてのメソッドがります。
3. いくつかのRESTfulなメソッドを定義してください。以上により、匿名フィールド、`MainController`は既に`Get`、`Post`、`delete`、`Put`およびその他のメソッドを持っていますので、これらは、ユーザが対応するリクエストが送られた時に呼ばれます（例えばPOSTを使用してリクエストをハンドルするために`Post`メソッドが呼び出されます）。それゆえ、`MainController`の`Get`メソッドをオーバーロードした後では、すべてのGETリクエストはの`beego.Controller`の代わりに`MainController`のメソッドを使用します。
4. `main`関数を定義してください。 C言語のようにGoでのすべてのアプリケーションは、エントリポイントとして`main`を使用しています。
5. `routes`へ登録してください。これは、特定の要求を担当するコントローラをbeegoに指示します。ここでは、 `/`に対しては`MainController`を登録しますので、`/`の全てのリクエストは`MainController`によってハンドルされます。最初の引数がパスであり、2番目は、コントローラへ登録したいポインタであることに注意してください。
6. デフォルトでは`8080`ポートでアプリケーションを実行します、`Ctrl`キーを押しながら`c`で終了します。

Windowsユーザのためのショートカットの`.bat`ファイルは次のとおりです:

`%GOPATH%/src`配下で、`step1.install-bee.bat`と`step2.new-beego-app.bat`ファイルを作成してください。

`step1.install-bee.bat`:

	set GOPATH=%~dp0..
	go build github.com\beego\bee
	copy bee.exe %GOPATH%\bin\bee.exe
	del bee.exe
	pause

`step2.new-beego-app.bat`:

	@echo Set value of APP same as your app folder
	set APP=coscms.com
	set GOPATH=%~dp0..
	set BEE=%GOPATH%\bin\bee
	%BEE% new %APP%
	cd %APP%
	echo %BEE% run %APP%.exe > run.bat
	echo pause >> run.bat
	start run.bat
	pause
	start http://127.0.0.1:8080

迅速にbeegoの作業を開始するために、これら2つのファイルをクリックしてください。そして、今後`run.bat`をただ実行してください。
