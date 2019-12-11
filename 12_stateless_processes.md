# Stateless Processes

Execute the app as one or more stateless processes

1. In rp1-music, add Redis dependencies.
```
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```

2. In application.properties change Http Session to use redis.
```
# Http Session
spring.session.store-type=redis
```

3. Start redis.
```
docker run --name rp1-redis -p 6379:6379 --network rp1-network -d redis
```

4. Run application and login.
```
./mvnw spring-boot:run
```

5. Kill application.

6. Rerun application and you should still be logged in after refresh.
```
./mvnw spring-boot:run
```

7. Run Redis cli.
```
docker run -it --network rp1-network --rm redis redis-cli -h rp1-redis
```

8. Query keys.
```
keys '*'
```

9. Kills session and refresh browser and you should be logged out.
```
del spring:session:sessions:<session id>
```