# Build, Release, Run/Design, Build, Release, Run

1. Create a Dockerfile at the root of the rp1recengine project.
```
FROM openjdk:8-jre

ENTRYPOINT ["java", "-jar", "/usr/share/rp1/rp1-rec-engine.jar"]

ARG JAR_FILE
ADD target/${JAR_FILE} /usr/share/rp1/rp1-rec-engine.jar
```

2. Add dockerfile-maven-plugin plugin to the build section of the pom.xml.
```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <id>default</id>
            <goals>
                <goal>build</goal>
                <goal>push</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>javajudd/${project.name}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

3. Build project and Docker image.
```
./mvnw package
```

4. Validate a Docker images was created.
```
docker images
```

5. Run Docker container from image.
```
docker run -it --rm -p 8080:8080 --network rp1-network javajudd/rp1-rec-engine:0.0.1-SNAPSHOT
```

6. Test by visiting [http://localhost:8080/api/v1/user/1/recommendation](http://localhost:8080/api/v1/user/2/recommendation)

7. Shut it down.