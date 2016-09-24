### アプリケーションのゼロダウンタイムアップデート

#### Autopilotプラグインのインストール

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

#### JREをアップデートしたbuildpackを指定

``` diff
  instances: 2
- buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.8.1
+ buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.9
  path: target/mrs-0.0.1-SNAPSHOT.jar
```

```
cf zero-downtime-push mrs -f manifest.yml
```
