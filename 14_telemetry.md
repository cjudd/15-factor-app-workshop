# Telemetry

1. Add the Actuator spring boot starter to the pom.xml.
```
    <dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
```

REPACKAGE and STOP and RM and RUN

2. Test the default actuator endpoints at http://localhost:9090/actuator/info and http://localhost:9090/actuator/health.

3. Test additional actuator endpoints such as http://localhost:9090/actuator/beans. This should fail since it is disabled by default for security reasons.

4. Enable the secure acutator endpoints by adding the following to the application.properties file.
```
# actuator
management.endpoints.web.exposure.include=*
```

5. Test remaining actuator endpoints.

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