## ソースコードの確認

「会議室予約システム」のソースコードをcloneしてください。

```
git clone https://github.com/making/mrs
```

「[Spring徹底入門](http://blt.ly/spring-book)」時点に比べて

* Spring Bootのバージョンが1.3から1.4
* データベースがPostgreSQLからMySQL (JDBCドライバはMariaDBのものを使用)

に[変更されています](https://github.com/making/mrs/commit/8394a53cc924ed6ca41fdb675c2b37fafc5db64f)。

PCF DevがMySQLしか同梱していないため、本ワークショップではMySQLを使用します。
