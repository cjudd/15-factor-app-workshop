# Dependencies/Dependency Management

1. In rp1recengine, edit pom.xml.

2. Add spring-boot-starter-data-neo4j dependency.
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```

3. Rebuild to download dependency.
```
./mvnw package
```