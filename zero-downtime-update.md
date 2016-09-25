## アプリケーションのゼロダウンタイムアップデート

### Autopilotプラグインのインストール

[Autopilot](https://github.com/contraband/autopilot/releases)をOSに合わせてダウンロードして、

```
cf install-plugin <ダウンロードしたファイル>
```


Macの場合、

``` console
$ chmod +x ~/Downloads/autopilot-darwin 
$ cf install-plugin ~/Downloads/autopilot-darwin 

**注意: プラグインは必ずしも信頼できない作成者によって書かれたバイナリーです。プラグインのインストールと使用は自らの責任で行ってください。**

プラグイン /Users/makit/Downloads/autopilot-darwin をインストールしますか? (y または n)> y

プラグイン autopilot-darwin をインストールしています...
OK
プラグイン autopilot v0.0.2 は正常にインストールされました。
```

### JREをアップデートしたbuildpackを指定

``` diff
 applications:
 - name: mrs
   memory: 680m
-  instances: 2
+  instances: 1
-  buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.8.1
+  buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.9
   path: target/mrs-0.0.1-SNAPSHOT.jar
```

メモリを768Mに上げたため、インスタンス数が2の場合、zero down time update中に最大2 x 2 x 680MB使用し、無償枠を超えてしまう。
そのため、ここではインスタンス数を1に減らしているが、これは本質的ではない。

```
cf zero-downtime-push mrs -f manifest.yml
```

> **注意**
> 
> `random-route: true`を設定していると、更新後のアプリケーションのURLが変わってします。`name`を一意にすべき。
