## アプリケーションメトリクスの集約

### コンテナのメトリクス

[PCF Metrics](https://metrics.run.pivotal.io/)の使い方は[こちら](https://github.com/Pivotal-Japan/cf-workshop/blob/master/pcf-metrics.md)。

### アプリケーションのメトリクス

Spring ActuatorのメトリクスをRedisに集約する。

`pom.xml`

``` xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

`MetricsConfig.java`

``` java
package mrs;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.autoconfigure.ExportMetricReader;
import org.springframework.boot.actuate.metrics.CounterService;
import org.springframework.boot.actuate.metrics.GaugeService;
import org.springframework.boot.actuate.metrics.aggregate.AggregateMetricReader;
import org.springframework.boot.actuate.metrics.export.MetricExportProperties;
import org.springframework.boot.actuate.metrics.reader.MetricReader;
import org.springframework.boot.actuate.metrics.repository.MetricRepository;
import org.springframework.boot.actuate.metrics.repository.redis.RedisMetricRepository;
import org.springframework.boot.actuate.metrics.writer.DefaultCounterService;
import org.springframework.boot.actuate.metrics.writer.DefaultGaugeService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;

@Configuration
public class MetricsConfig {

	@Autowired
	RedisConnectionFactory redisConnectionFactory;

	@Autowired
	MetricExportProperties export;

	// Metrics for an instance
	@Bean
	@ExportMetricReader
	MetricRepository metricRepository() {
		return new RedisMetricRepository(redisConnectionFactory,
				export.getRedis().getPrefix(), export.getRedis().getKey());
	}

	@Bean
	GaugeService gaugeWriter() {
		return new DefaultGaugeService(metricRepository());
	}

	@Bean
	CounterService counterService() {
		return new DefaultCounterService(metricRepository());
	}

	// Metrics for all instances
	@Bean
	MetricRepository aggregateMetricRepository() {
		return new RedisMetricRepository(redisConnectionFactory,
				export.getRedis().getAggregatePrefix(), export.getRedis().getKey());
	}

	@Bean
	@ExportMetricReader
	MetricReader aggregateMetricReader() {
		return new AggregateMetricReader(aggregateMetricRepository());
	}

}
```

`AdminSecurityConfig.java`

``` java
package mrs;

import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;

@EnableWebSecurity
@Order(-5)
public class AdminSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.antMatcher("/admin/**").authorizeRequests().antMatchers("/admin/**")
				.hasRole("OPERATOR").and().httpBasic().and().sessionManagement()
				.sessionCreationPolicy(SessionCreationPolicy.STATELESS).and().csrf()
				.disable();
	}

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().withUser("admin").password("admin")
				.roles("OPERATOR");
	}
}
```

`application.properties`

``` diff
  logging.level.org.hibernate.SQL=DEBUG
  logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
+ management.context-path=/admin
+ spring.application.name=mrs
```

512MBだと少し足りないので、メモリを680MBに増やす。

`manifest.yml`

``` diff
 applications:
 - name: mrs
-  memory: 512m
+  memory: 680m
   instances: 2
```

```
./mvnw clean package -DskipTests=true
cf push
```

![](https://qiita-image-store.s3.amazonaws.com/0/1852/2f173442-4dd0-9505-8d9f-2df8623779c4.png)
