## セッションのクラスタ化

* [VI. プロセス](https://12factor.net/ja/processes)

### Spring Sessionの依存関係追加

``` xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session</artifactId>
            </dependency>
        <dependency>
```

### ローカル環境用の設定

ローカル開発時にはRedisバックエンドにしない。

`application.properties`

``` diff
  spring.jpa.properties.hibernate.format_sql=true
  spring.datasource.sql-script-encoding=UTF-8
+ spring.session.store-type=hash_map
  logging.level.org.hibernate.SQL=DEBUG
  logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

#### クラウド環境の設定

クラウド上でRedisバックエンドに。

`application-cloud.properties`

``` diff
  spring.jpa.hibernate.ddl-auto=none
+ spring.session.store-type=redis
```

`RedisConnectionFactory`はアプリケーションにバインドされたサービスインスタンスに接続できるように作る。

`CloudConfig.java`

``` diff
  package mrs;

  import org.springframework.cloud.config.java.AbstractCloudConfig;
  import org.springframework.cloud.service.PooledServiceConnectorConfig;
  import org.springframework.cloud.service.relational.DataSourceConfig;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Profile;
+ import org.springframework.data.redis.connection.RedisConnectionFactory;

  import javax.sql.DataSource;

  @Configuration
  @Profile("cloud")
  public class CloudConfig extends AbstractCloudConfig {

  	@Bean
  	DataSource dataSource() {
  		return connectionFactory().dataSource(new DataSourceConfig(
  				new PooledServiceConnectorConfig.PoolConfig(0, 2, 3000), null));
  	}

+ 	@Bean
+ 	RedisConnectionFactory redisConnectionFactory() {
+ 		return connectionFactory().redisConnectionFactory();
+ 	}

  }
```

本質的ではないが、Redis Cloudを使うと[CONFIGコマンドを使えない問題](https://github.com/spring-projects/spring-session/issues/124)が起こるので、次の設定を追加。(`CloudConfig`に設定しても可)

`MrsApplication.java`

``` diff
  package mrs;

  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
+ import org.springframework.context.annotation.Bean;
+ import org.springframework.session.data.redis.config.ConfigureRedisAction;

  @SpringBootApplication
  public class MrsApplication {

  	public static void main(String[] args) {
  		SpringApplication.run(MrsApplication.class, args);
  	}

+ 	@Bean
+ 	public static ConfigureRedisAction configureRedisAction() {
+ 		return ConfigureRedisAction.NO_OP;
+ 	}
  }
```

### Redisサービスインスタンス作成

Pivotal Web Servicesの場合

```
cf create-service rediscloud 30mb  mrs-redis
```

Redis Cloudの`30mb`プラン(free)は1Organizationあたり1インスタンスしか作れないので注意。

PCF Devの場合

```
cf create-service p-redis shared-vm mrs-redis
```

### サービスインスタンスのバインド

`manifest.yml`

``` diff

 applications:
 - name: mrs-<yourname>
   memory: 512m
   instances: 2
   buildpack: https://github.com/cloudfoundry/java-buildpack.git#v3.8.1
   path: target/mrs-0.0.1-SNAPSHOT.jar
   services:
   - mrs-db
   - mrs-log
+  - mrs-redis
   env:
     SPRING_DATASOURCE_INITIALIZE: false
```


```
./mvnw clean package -DskipTests=true
cf push
```

> **ノート**
> 
> 以下でもOK
>
> ```
> cf bind-services mrs-<yourname> mrs-redis
> cf restage mrs-<yourname>
> ```


片方のサービスインスタンスがクラッシュしても、再ログインすることなくもう片方のインスタンスでサービスを続行できる。
また、アプリケーションを再デプロイしてもログインし直す必要がなくなる。
