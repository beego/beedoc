---
root: true
name: Install / Upgrade
sort: 1
---

# beegoのインストール

従来のGoの方法でbeegoをインストール可能です:

	go get github.com/astaxie/beego

よくある質問:

- gitのがインストールされていません。お使いのシステムによったgitをインストールしてください。
- gitがHTTPSぶアクセスできません。ローカルのgitのコンフィグを設定し、HTTPSのバリデーションを閉じてください:

		git config --global http.sslVerify false

- オフラインでbeegoをインストールするには？今や良い解決策がありますリリースごとにダウンロードしてインストールするためのパッケージを作成しています。

# beegoのアップグレード

Goのコマンドからbeegoをアップグレード、または、ソースコードからアップグレードすることができます。

- Goのコマンドから: この方法でbeegoのアップグレードすることを推奨しています:

		go get -u github.com/astaxie/beego

- ソースコードから: `https://github.com/astaxie/beego`からソースコードをダウンロードして下さい。コピーして`$GOPATH/src/github.com/astaxie/beego`へ上書きしてください。そうしたら、`go install`を実行してbeegoをアップグレードしてください:

		go install 	github.com/astaxie/beego

**1.0以前のアップグレードに関して:** 1.0以降のbeegoのAPIはすでに安定しています。基本的にすべてのアップグレードと互換性があります。1.0より低いバージョンをまだ使用されている場合は、最新のAPIに基づいていくつかのメソッドやパラメータを変更する必要があります。
