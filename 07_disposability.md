# Disposability

Maximize robustness with fast startup and graceful shutdown

1. Run the app to see how long it takes to start up.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar
```

2. Edit application.propeties file to disable ASCII art on start up.
```
# speed up start up
spring.main.banner-mode=off
```

3. Build.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar
```

4. Run the app to see how long it takes to start up.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar
```

5. Run app in debug mode to determine unnecessary auto config on startup.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar --debug
```

5. Disable unnessary auto config in application.properties file.
```
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration
```

6. Build.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar
```

7. Run the app to see how long it takes to start up.
```
java -jar target/rp1music-0.0.1-SNAPSHOT.jar
```