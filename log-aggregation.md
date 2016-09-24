### アプリケーションのスケールアウト

* [XI. ログ](https://12factor.net/ja/logs)


#### ログの確認

```
cf logs mrs --recent
```

追跡

```
cf logs mrs
```

#### 外部に転送


[こちら](https://github.com/Pivotal-Japan/cf-workshop/blob/master/logging.md)を参照して、[Logit.io](https://logit.io/)または[Papertrail](https://papertrailapp.com/)のアカウントを作成して、

```
cf create-user-provided-service mrs-log -l syslog://....
```




`manifest.yml`

``` diff
  path: target/mrs-0.0.1-SNAPSHOT.jar
   services:
   - mrs-db
+  - mrs-log
  env:
    SPRING_DATASOURCE_INITIALIZE: false
```

```
cf push
```
