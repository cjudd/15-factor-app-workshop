# Configuration, Credentials, and Code

1. Build rp1-music Docker image.
```
../rp1-music
./mvnw package
```

2. Validate javajudd/rp1-music Docker images was created.
```
docker images
```

3. Run Docker container from rp1-music image.
```
docker run --name rp1-music -p 8080:8080 --network rp1-network -it --rm javajudd/rp1-music:0.0.1-SNAPSHOT
```

Why does it throw an exception? It throws a "java.net.ConnectException: Connection refused" exception because inside the embedded application.properties file it uses the spring.datasource.url property that is pointing to a MySQL database at localhost. However the container is not running a MySQL database. So we have to externally configure it to point to the MySQL database running in the rp1-music-db container.

4. Rerun the Docker container using an environment variable to configure it.
```
docker run --name rp1-music -p 8080:8080 --network rp1-network -e SPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 -e SPRING_DATASOURCE_USERNAME=rp1 -e SPRING_DATASOURCE_PASSWORD=rp1+1 -it --rm javajudd/rp1-music:0.0.1-SNAPSHOT
```