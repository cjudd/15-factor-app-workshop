```
git init
git add .
git commit -m "Created a Spring Boot 2.0.1 app using start.spring.io using Web, JPA, MySQL, Thymeleaf and Flyway dependencies."

docker run --name rp1-music-db -e MYSQL_ROOT_PASSWORD=root+1 -e MYSQL_DATABASE=rp1 -e MYSQL_USER=rp1 -e MYSQL_PASSWORD=rp1+1 -p 3306:3306 -d mysql:latest


docker run --name rp1-neo4j -p 7474:7474 -p 7687:7687 -d neo4j:latest
```


docker run --rm -it -p 7474:7474 -p 7687:7687 neo4j:latest
