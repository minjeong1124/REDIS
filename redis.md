### 개념

(**RE**mote **DI**ctionary **S**erver)
in memory 키 - 값 데이터 구조 스토어 (NoSQL DBMS 로 분류 되기도 함. 메모리에 상주 하면서 RDBMS의 캐시 솔루션으로 주로 사용 됨)

메모리를 이용하여 고속(RDBMS의 구조상 DISK에서 데이터를 꺼내는 데 Memory에서 읽어 들이는 것보다 천 배 가량 더 느리기 때문)으로 <key, value> 스타일의 데이터를 저장하고 불러올 수 있는 원격 시스템


### 장점

1. 빠른 성능
- 데이터를 디스크 / SSD에 저장하는 대부분의 데이터베이스 관리 시스템과 달리 모든 Redis 데이터는 서버의 메인 메모리에 저장 됨. 디스크에 액세스 해야 할 필요 없앰으로써 검색 시간으로 인한 지연 방지, CPU 명령을 적게 사용하는 좀 더 간단한 알고리즘으로 데이터에 액세스 할 수 있음. 일반적으로 작업 실행하는 데 1ms 미만 소요 됨, 키-값 구조라 쿼리 따로 필요 없이 데이터 바로 가져올 수 있음
2. 인 메모리 데이터 구조
- 사용자가 다양한 데이터 유형에 매핑 되는 키 저장할 수 있음. 기본적인 데이터 유형은 String
3. 복제 및 지속성
- master, slave 아키텍처를 사용하며 데이터가 여러 slave server에 복제 될 수 있음. 메인 서버에 장애가 발생하는 경우 여러 서버로 요청이 분산 될 수 있음
안정성을 제공하기 위해 특정 시점 SNAPSHOT과 데이터가 변경 될 때마다 이를 디스크에 저장하는 (AOF) 생성 모두 지원. (장애 발생 시 Redis 데이터 복원 할 수 있음)
- Snapshot : 스냅샷은 어떤 특정 시점의 데이터를 DISK에 옮겨 담는 방식을 뜻합니다.
- AOF : Redis의 모든 write/update 연산 자체를 모두 log 파일에 기록하는 형태입니다. 서버가 재시작할 시 write/update를 순차적으로 재실행, 데이터를 복구합니다.
- redis 공식 문서에서의 권장 사항은 RDBMS의 rollback 시스템같이 두 방식을 혼용해서 사용하는 것입니다. 주기적으로 snapshot으로 backup 하고 다음 snapshot 까지의 저장을 AOF 방식으로 수행하는 것
4. 다양성
- String, List, Hashes, Set, SortedSet(with Score) 등 여러 형식의 자료 구조 지원
참고 - [https://ozofweird.tistory.com/entry/Redis-Redis-데이터-유형?category=895904?category=895904](https://ozofweird.tistory.com/entry/Redis-Redis-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%9C%A0%ED%98%95?category=895904?category=895904)



### Redis commands

[https://redis.io/commands#hash](https://redis.io/commands#hash)

### Redis 사용

- **k8s W/Redis**

- **Spring Boot w/Redis**

[https://ssoco.tistory.com/19](https://ssoco.tistory.com/19)

1. dependency 

```jsx
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

[Redis client - Jedis vs Lettuce 비교글](https://jojoldu.tistory.com/418)

2. properties (local)


```jsx
### Redis --------------------------------------------------------------------------------------------------------------------------------------------------------------------
spring.redis.lettuce.pool.max-active=10
spring.redis.lettuce.pool.max-idle=10
spring.redis.lettuce.pool.min-idle=2
spring.redis.port=6379
spring.redis.host={redis외부IP}
```

3. config

```jsx
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        // key는 일반 문자열을 쓸테니까 StringRedisSerializer 로 지정
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(Config.class));
				// GenericJackson2JsonRedisSerializer()로 셋팅하는 부분은 데이터 형태를 하나의 표준화 된 VO형태가 아니라 여러가지 형태로 공존
				// 나는 Config entity만 사용할 것이므로 Jackson2JsonRedisSerializer<>(Config.class)
        return redisTemplate;
    }

}
```
