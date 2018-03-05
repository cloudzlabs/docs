---
date: "2018-03-05T13:10:36+09:00"
title: "Redis Overview(with Spring Boot)"
authors: ["hunkee1017"]
categories:
  - posts
tags:
  - Spring Boot
  - Redis
description: "Spring Boot 환경에서 Redis를 사용하는 방법"
draft: true
---

**Redis**는 Key/Value형태의 In-Memory 데이터베이스로서 데이터 저장 및 캐싱, 그리고 메시지 브로커로 널리 사용 되고 있습니다.

**Redis**의 가장 큰 특징으로는 In-Memory 데이터베이스가 가지는 장점인 처리속도가 굉장히 빠르다는 점이 있습니다. 물론 이것 자체는 휘발성 데이터이기 때문에 케이스에 따라서 데이터를 덤프하는 스케쥴링을 통해 정합성 및 영속성을 관리할 수 있습니다. 하지만 이것도 불안하다 싶어 조금 더 나아간다면 고가용성을 보장하기 위해 Master-Slave 구조의 아키텍쳐링까지 설계할 수 있습니다. 

또한, **Redis**의 내부를 열어보면 데이터 스트럭쳐로 String, Hash, List, Sets, Sorted Sets등 기본적인 쿼리 뿐만 아니라 Bitmaps, Hyperloglogs, Geospatial indexes와 같은 Radius 쿼리까지도 지원하는 등 강력한 기능이 있습니다.

Pivotal에서도 강력한 **Redis**의 기능을 사용하기 위해 Spring Boot를 통해 손쉽게 여러 프로젝트와 연결할 수 있는 방법을 가이드 하고 있습니다.

**지금부터는 이런 Redis를 Spring Boot에서 사용하는 방법을 알아보겠습니다.**

> Spring Boot, Maven를 사용해서 작성했습니다.

### Redis Dependency 추가

* * *

Redis를 Spring-boot에서 사용하기 위해서는 가장 먼저 Redis의 Dependency를 추가해야합니다.

>pom.xml에 다음과 같은 설정을 추가함

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```


### Redis Session

* * *

Redis는 Spring session과 연동하여 Session에 대한 저장소를 Redis로 관리할 수 있습니다.

*Dependency*

> pom.xml에 Spring Session을 사용하기 위한 정보를 넣습니다.

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
```

*Annotation*

> 해당 Spring Session을 Redis로 사용하겠다는 annotation을 넣습니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

@SpringBootApplication
@EnableRedisHttpSession // Redis를 세션형태로 사용하겠다는 설정
public class RedisApplication {

	public static void main(String[] args) {
		SpringApplication.run(RedisApplication.class, args);
	}
}
```

*Property*

> 프로퍼티에서 session의 Store-type을 redis로 설정해줍니다.

```yml
spring:
  session:
    store-type: redis
```

*결과*

> $redis-cli> keys spring:session*

```
"spring:session:sessions:expires:e7e5a693-54b9-489f-b8c2-25c993de7ec9"
"spring:session:expirations:1520229840000"
"spring:session:sessions:e7e5a693-54b9-489f-b8c2-25c993de7ec9"
```    

  

### Redis Cache

* * *

Redis Cache는 Spring Cache를 Redis에서 처리 할 수 있도록 지원합니다.

*Dependency*
    
> pom.xml에 캐시를 사용하기 위한 Dependency를 추가합니다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

*Annotation*

> 해당 Spring Cache을 Redis로 사용하겠다는 annotation을 넣습니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class RedisApplication {

	public static void main(String[] args) {
		SpringApplication.run(RedisApplication.class, args);
	}
}
```

*Property*

```yaml
spring:
  cache:
    type: redis
```    
    
*사용하기*

```java 
// 사용할 Cache에 대한 configuration
@CacheConfig(cacheNames="cacheName")

// 캐싱을 할 대상 Method에 붙여주는 Annotation
@Cacheable

// 캐시된 대상을 비워줄 Method 위에 붙여주는 Annotation
@CacheEvict(cacheNames="cacheName")
```
  

### Redis Transaction

* * *

레디스에서 데이터를 저장할때 트랜잭션을 통해 처리할 수 있도록 설정합니다.

주요 명령어는 [MULTI](https://redis.io/commands/multi) , [EXEC](https://redis.io/commands/exec) , [DISCARD](https://redis.io/commands/discard), [WATCH](https://redis.io/commands/watch)

*   MULTI : 트랜잭션 블록의 시작을 표시. 이 구문 안에 포함된 구문들은 전부 Queue로 넣어진 후 EXEC를 만날 때까지 대기.
*   EXEC : 이전에 대기중인 모든 명령을 트랜잭션에서 실행 하고 연결 상태를 정상으로 복원.
*   DISCARD : 이전에 대기중인 모든 명령을 트랜잭션에서 플러시 해버리고 연결 상태를 정상으로 복원.
*   WATCH : 트랜잭션의 조건부 실행을 감시하는 지정된 키를 마크. 이 watch가 걸려있는 key를 외부에서 접근하려고 시도하면 Discard를 발생시킴.

  

1) Redis Transaction

> 명시적으로 사용하기 위해서 커스터마이징을 수행

```java
// execute a transaction
List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
  public List<Object> execute(RedisOperations operations) throws DataAccessException {
    operations.multi();
    operations.opsForSet().add("key", "value1");

    // This will contain the results of all ops in the transaction
    return operations.exec();
  }
});
System.out.println("Number of items added to set: " + txResults.get(0));
```
  

2) @Transactional Support

> Framework의 Transaction을 사용해서 처리

```java
/* Sample Configuration */
@Configuration
public class RedisTxContextConfiguration {
  @Bean
  public StringRedisTemplate redisTemplate() {
    StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
    // explicitly enable transaction support
    template.setEnableTransactionSupport(true);
    return template;
  }

  @Bean
  public PlatformTransactionManager transactionManager() throws SQLException {
    return new DataSourceTransactionManager(dataSource());
  }

  @Bean
  public RedisConnectionFactory redisConnectionFactory( // jedis, lettuce, srp,... );

  @Bean
  public DataSource dataSource() throws SQLException { // ... }
}
```

1. [https://redis.io/topics/transactions](https://redis.io/topics/transactions)
2. [https://docs.spring.io/spring-data/redis/docs/1.8.6.RELEASE/reference/html/#redis:pubsub:subscribe](https://docs.spring.io/spring-data/redis/docs/1.8.6.RELEASE/reference/html/#redis:pubsub:subscribe)

  

  

### Redis Persistence

* * *

레디스 데이터의 영속성을 보장하기 위해 Data를 백업하고 관리하는 방법입니다.
Redis홈페이지에서 권장하는 방법은 **RDB와 AOF를 모두 사용**하여 데이터 정합성을 보장하는 것입니다.

1) RDB 방식

*   **dump.rdb** 이름으로 저장
*   **장점**
    *   원하는 시점에서 복구데이터를 남길 수 있습니다.
    *   재해 복구에 매우 유용.
    *   RDB는 AOF에 비해 큰 데이터 세트를 사용하여 재시작을 빠르게 수행.
    
*   **단점**
    *   Redis가 작동을 멈춘 경우(예 : 정전 후)에 데이터 손실 가능성을 최소화해야하는 경우 RDB가 좋지 않습니다. (예를 들어 데이터 세트에 대해 최소 5 분 및 100 회 기록 후, 여러 저장 지점을 가질 수 있음).
    *   RDB는 자식 프로세스를 사용하여 디스크에 저장하기 위해 fork ()를 자주 수행 해야 합니다. 데이터 세트가 크면 Fork ()에 시간이 많이 걸릴 수 있으며 CPU 성능이 좋지 않은 경우 Redis는 클라이언트에 수 밀리 초 또는 1 초간 서비스를 제공하지 않을 수 있습니다.

* *Synchronously 백업*

    ```shell
    $ src/redis-cli
      127.0.0.1:6379> help save
        SAVE -
        summary: Synchronously save the dataset to disk
        since: 1.0.0
        group: server

      127.0.0.1:6379> save
    ```
    
* *Background 백업*

    ```shell
    $ src/redis-cli
      127.0.0.1:6379> help bgsave
        BGSAVE -
        summary: Asynchronously save the dataset to disk
        since: 1.0.0
        group: server

      127.0.0.1:6379> bgsave
    ```
    

2) AOF 방식

*   **appendonly.aof** 이름으로 저장됩니다.
*   **장점**
    *   AOF Redis는 매 초 fsync를 수행하여 데이터 정합성에서 유리하다(1 초의 쓰기 정도만 손실 할 수 있습니다)
    *   AOF 로그는 추가 전용 로그이므로 전원이 중단 될 경우 손상 문제가 없습니다. 어떤 이유로 (디스크가 가득 차거나 다른 이유로) 로그가 절반으로 작성된 명령으로 끝나더라도 redis-check-aof 도구로 쉽게 수정할 수 있습니다.
    *   Redis는 너무 커지면 백그라운드에서 AOF를 자동으로 다시 쓸 수 있어 데이터의 유실이 최소화 됩니다.
    *   AOF에는 이해하기 쉽고 구문 분석 할 수있는 형식으로 차례로 모든 작업의 로그가 포함되어 있습니다. (예를 들어, FLUSHALL 명령을 사용하여 모든 오류를 플러시 했더라도 로그를 다시 작성하지 않으면 서버를 중지하고 최신 명령을 제거한 다음 Redis를 다시 시작하여 데이터 세트를 저장할 수 있습니다.)

*   **단점**
    *   AOF 파일은 대개 동일한 데이터에 해당하는 RDB 파일보다 용량이 큽니다.
    *   AFS는 정확한 fsync 정책에 따라 RDB보다 느릴 수 있습니다.
    *   AOF로 인해 다시로드 할 때 정확히 동일한 데이터 세트가 재현되지 않을 가능성이 있습니다.
    
*   *서버 실행할 때 AOF 활성*
    ```shell
    $ src/redis-server --appendonly yes
    ```
*   *Redis-cli에서 AOF 활성*
    ```shell
    $ src/redis-cli
      127.0.0.1:6379> help BGREWRITEAOF
        BGREWRITEAOF -
        summary: Asynchronously rewrite the append-only file
        since: 1.0.0
        group: server

      127.0.0.1:6379> BGREWRITEAOF
    ```

참조 :

[https://redis.io/topics/persistence](https://redis.io/topics/persistence)