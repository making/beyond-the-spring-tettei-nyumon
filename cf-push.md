## Cloud Foundryにデプロイ

### Cloud Foundryにログイン

Pivotal Web Servicesの場合


```
cf login -a https://api.run.pivotal.io -u <メールアドレス> -p <パスワード>
```

PCF Devの場合

```
cf login -a https://api.local.pcfdev.io --skip-ssl-validation -u admin -p admin
```


### ビルド


```
./mvnw clean package -DskipTests=true
```

### MySQLサービスの作成

[IV. バックエンドサービス](https://12factor.net/ja/backing-services) in 12 Factor


Pivotal Web Servicesの場合

```
cf create-service cleardb spark mrs-db
```

PCF Devの場合

```
cf create-service p-mysql 512mb mrs-db
```

### デプロイ

`manifest.yml`作成

``` yml
applications:
- name: mrs-<yourname> # <--- !! Change !!
  memory: 512m
  instances: 1
  buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.8.1
  path: target/mrs-0.0.1-SNAPSHOT.jar
  services:
  - mrs-db
  env:
    SPRING_DATASOURCE_INITIALIZE: false
```


```
cf push --no-start
cf set-env mrs-<yourname> SPRING_DATASOURCE_INITIALIZE true # 初回のみ
cf start mrs-<yourname>
cf set-env mrs-<yourname> SPRING_DATASOURCE_INITIALIZE false
```

* [VI. プロセス](https://12factor.net/ja/processes)
* [VII. ポートバインディング](https://12factor.net/ja/port-binding)
* [XII. 管理プロセス](https://12factor.net/ja/admin-processes)

in 12 Factor

Pivotal Web Servicesの場合

`http://mrs-<yourname>.cfapps.io`

PCF Devの場合

`http://mrs-<yourname>.local.pcfdev.io`

### Spring Cloud Connectorsを使う

[III. 設定](https://12factor.net/ja/config) in 12 Factor

``` xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-spring-service-connector</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-cloudfoundry-connector</artifactId>
        </dependency>
```


``` java
package mrs;

import org.springframework.cloud.config.java.AbstractCloudConfig;
import org.springframework.cloud.service.PooledServiceConnectorConfig;
import org.springframework.cloud.service.relational.DataSourceConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

import javax.sql.DataSource;

@Configuration
@Profile("cloud")
public class CloudConfig extends AbstractCloudConfig {

	@Bean
	DataSource dataSource() {
		return connectionFactory().dataSource(new DataSourceConfig(
				new PooledServiceConnectorConfig.PoolConfig(0, 2 /* cleardb sparkプラン用 */, 3000), null));
	}
}
```
