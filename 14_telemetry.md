# Telemetry

1. Add the Actuator spring boot starter to the pom.xml.
```
    <dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
```

2. Rebuild and redeploy container.
```
./mvnw package
docker stop rp1-api
docker rm rp1-api
docker run --name rp1-api -d --network rp1-network -p9090:8080 -eSPRING_DATA_NEO4J_URI=bolt://rp1-neo4j -eSPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 javajudd/rp1-rec-engine:0.0.1-SNAPSHOT
```

3. Test the default actuator endpoints at http://localhost:9090/actuator/info and http://localhost:9090/actuator/health.

4. Test additional actuator endpoints such as http://localhost:9090/actuator/beans. This should fail since it is disabled by default for security reasons.

5. Rebuild and redeploy container.
```
./mvnw package
docker stop rp1-api
docker rm rp1-api
docker run --name rp1-api -d --network rp1-network -p9090:8080 -eSPRING_DATA_NEO4J_URI=bolt://rp1-neo4j -eSPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 javajudd/rp1-rec-engine:0.0.1-SNAPSHOT
```

6. Enable the secure acutator endpoints by adding the following to the application.properties file.
```
# actuator
management.endpoints.web.exposure.include=*
```

7. Test remaining actuator endpoints.

http://localhost:9090/actuator/auditevents

http://localhost:9090/actuator/beans

http://localhost:9090/actuator/conditions

http://localhost:9090/actuator/configprops

http://localhost:9090/actuator/env

http://localhost:9090/actuator/flyway

http://localhost:9090/actuator/heapdump

http://localhost:9090/actuator/logfile

http://localhost:9090/actuator/loggers

http://localhost:9090/actuator/metrics

http://localhost:9090/actuator/prometheus

http://localhost:9090/actuator/scheduledtasks

http://localhost:9090/actuator/sessions

http://localhost:9090/actuator/shutdown

http://localhost:9090/actuator/threaddump