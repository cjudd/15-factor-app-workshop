# Concurrency

Scale out via the process model

1. Start multiple rp1-music apps on different ports.
```
docker run --name rp1-music-1 -p 8080:8080 --network rp1-network -e SPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 -e SPRING_DATASOURCE_USERNAME=rp1 -e SPRING_DATASOURCE_PASSWORD=rp1+1 -e SPRING_OUTPUT_ANSI_ENABLED=NEVER -eSPRING_REDIS_HOST=rp1-redis --log-driver=syslog --log-opt syslog-address=tcp://:5000 --log-opt syslog-facility=daemon -d javajudd/rp1-music:0.0.1-SNAPSHOT

docker run --name rp1-music-2 -p 8081:8080 --network rp1-network -e SPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 -e SPRING_DATASOURCE_USERNAME=rp1 -e SPRING_DATASOURCE_PASSWORD=rp1+1 -e SPRING_OUTPUT_ANSI_ENABLED=NEVER --log-driver=syslog -eSPRING_REDIS_HOST=rp1-redis --log-opt syslog-address=tcp://:5000 --log-opt syslog-facility=daemon -d javajudd/rp1-music:0.0.1-SNAPSHOT
```

2. To configure HAProxy, in a seperate directory create an haproxy.cfg file and add the following:
```
global  
        debug  
  
defaults  
        log global  
        mode    http  
        timeout connect 5000  
        timeout client 5000  
        timeout server 5000  
  
frontend main  
        bind *:80  
        default_backend app  
  
backend app  
        balance roundrobin  
        mode http  
        server srv1 rp1-music-1:8080
        server srv2 rp1-music-2:8080
```

3. Start HAProxy.
```
docker run -d --name rp1-haproxy -p 80:80 --network rp1-network -v ${PWD}:/usr/local/etc/haproxy:ro haproxy:1.7
```

4. Visit http://localhost.

5. Start another rp1-music app.
```
docker run --name rp1-music-3 -p 8082:8080 --network rp1-network -e SPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 -e SPRING_DATASOURCE_USERNAME=rp1 -e SPRING_DATASOURCE_PASSWORD=rp1+1 -e SPRING_OUTPUT_ANSI_ENABLED=NEVER --log-driver=syslog -eSPRING_REDIS_HOST=rp1-redis --log-opt syslog-address=tcp://:5000 --log-opt syslog-facility=daemon -d javajudd/rp1-music:0.0.1-SNAPSHOT
```

6. Add new server to HAPRoxy.
```
        server srv3 rp1-music-3:8080
```

7. Reread HAProxy config.
```
docker kill -s HUP rp1-haproxy
```

