---
name: ライブモニタ
sort: 1
---

# ライブモニタ

以前ツールボックスモジュールについてお話しました。アプリケーションが実行されている場合、デフォルトで`127.0.0.1:8088`をリスンします。インターネットからアクセスすることはできませんが、nginxのプロキシのような他の方法によってアクセスすることができます。

>>> For security reason it is recommend to block 8088 in firewall.

Monitorはデフォルトでは非表示です。こうすることで表示できます:

	beego.EnableAdmin = true
	
リスンするポートを変えることもできます:

	beego.AdminHttpAddr = "localhost"
	beego.AdminHttpPort = 8888
	
Webブラウザを開き、`http://localhost:8088/`にアクセスしてください、`Welcome to Admin Dashboard`が表示されていると思います。

今は最初のバージョンです。我々はそれを開発し続けています。

## リクエストの統計情報

`http://localhost:8088/aps`にアクセスすれば表示されます:

	| requestUrl                                        | method     | times            | used             | max used         | min used         | avg used         |
	| /                                                 | GET        |  2               | 2.35ms           | 1.30ms           | 1.04ms           | 1.17ms           |
	| /favicon.ico                                      | GET        |  1               | 79.30us          | 79.30us          | 79.30us          | 79.30us          |
	| /src/xx                                           | GET        |  1               | 923.09us         | 923.09us         | 923.09us         | 923.09us         |
	| /src                                              | GET        |  1               | 792.93us         | 792.93us         | 792.93us         | 792.93us         |
	| /123                                              | GET        |  1               | 906.04us         | 906.04us         | 906.04us         | 906.04us         |

## パフォーマンスのプロファイリング

プロファイリングのためのいくつかのパラメータがあります。`http://localhost:8088/prof`にアクセスし、以下のパラメータによりさまざまな情報を得ることができます。

	'/prof?command=lookup goroutine'のようなURLへリクエストしてください
        コマンドは、以下の種類があります:
	1. lookup goroutine
	2. lookup heap
	3. lookup threadcreate
	4. lookup block
	5. start cpuprof
	6. stop cpuprof
	7. get memprof
	8. gc summary

## ヘルスチェック

ヘルスチェックを確認するために、`http://localhost:8088/healthcheck`からヘルスチェックのロジックを手動で登録する必要があります

## タスク

アプリケーション内でタスクを追加し、タスクのステータスやトリガーを手動でチェックできます。

- タスクのステータスのチェック: `http://localhost:8088/task`
- タスクを手動で実行: `http://localhost:8088/runtask?taskname=task_name`

## コンフィグのステータス

アプリケーションの開発の後、実行されているアプリケーションのコンフィグも知りたいことがあります。beegoのモニタにもこの機能が提供されています。

- すべてのコンフィグを覧る: `http://localhost:8088/listconf?command=conf`
- すべてのルートを見る: `http://localhost:8088/listconf?command=router`
- すべてのフィルタを見る: `http://localhost:8088/listconf?command=filter`
