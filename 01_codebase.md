# Codebase/One codebase, one application

1. Start MySQL database (See [Backing Services](08_backing_services.md)).

```
docker run --name rp1-music-db -e MYSQL_ROOT_PASSWORD=root+1 -e MYSQL_DATABASE=rp1 -e MYSQL_USER=rp1 -e MYSQL_PASSWORD=rp1+1 -p 3306:3306 -d mysql:latest
```

2. Clone the existing repository.
```
git clone https://github.com/cjudd/rp1-music
cd rp1-music
```

3. Build & Run application.
```
./mvnw spring-boot:run
```

4. Browse to [http://localhost:8080](http://localhost:8080) and login with admin/password1, user1/password1 or user2/password1.
