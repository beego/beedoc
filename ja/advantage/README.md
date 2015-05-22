---
root: true
name: Advanced Beego
sort: 6
---

# Advanced Beego
＃高度なビーゴ


We have demonstrated the basic use of beego. Now we will talk about more advanced topics.
beegoの基本的な使用を実証しました。では、より高度なトピックについてお話します。

- [In process monitor](./monitor.md)

  Beego will serve at two ports. One is 8080 for application to serve users. Another is 8088, to monitor the process status, to execute tasks and so on.
  beegoは、2つのポートに配信されます。 1つは、アプリケーションがユーザーにサービスを提供するための8080です。もう一つは、プロセスのステータスを監視するようにタスクを実行すると、 `8088`です。
	
- [Filters](../mvc/controller/filter.md)

  Filters is a very convenient feature for you to extend your logic. You can easily implement user authentication, log visiting, compatibility switching and so on.
フィルタはロジックを拡張するための非常に便利な機能です。簡単に互換性が切り替えなど、訪問ログオン、ユーザー認証を実装することができます。	

- [Reload](./reload.md)

  Reload is always mentioned in web development that allows deploying application without interrupt user requests.
リロードは、常に割り込みユーザ要求せずにアプリケーションをデプロイできるWeb開発に記載されています。

>>> This feature is not well done yet. It only tested on Mac and Linux. It haven't been tested on production environment yet. It's still under testing, so take your own risk to use it. It's recommended to use upstream of nginx.
>>>この機能は、よく行われていません。これは、MacおよびLinux上でテストされています。これは、まだ本番環境でテストされていません。これは、テストの下ではまだですので、それを使用するように自分の責任を取ります。これは、 nginxのの上流を使用することをお勧めします。

