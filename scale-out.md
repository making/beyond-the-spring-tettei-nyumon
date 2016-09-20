### アプリケーションのスケールアウト

* [VI. プロセス](https://12factor.net/ja/processes)
* [VII. ポートバインディング](https://12factor.net/ja/port-binding)
* [VIII. 並行性](https://12factor.net/ja/concurrency)


`src/main/resources/templates/rooms/listRooms.html`

``` diff
 <body>
 <h3>会議室</h3>		 <h3>会議室</h3>
+<h4>Index <span th:text="${T(java.lang.System).getenv('CF_INSTANCE_INDEX')}"></span></h4>
 <a th:href="@{'/rooms/' + ${date.minusDays(1)}}">&lt; 前日</a>		 <a th:href="@{'/rooms/' + ${date.minusDays(1)}}">&lt; 前日</a>
```

`manifest.yml`

``` diff
memory: 512m
-instances: 1
+instances: 2
```

```
./mvnw clean package -DskipTests=true
cf push
```
