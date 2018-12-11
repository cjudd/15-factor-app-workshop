# Backing Services

Treat backing services as attached resources

1. Start neo4j database.
```
docker run --name rp1-neo4j --network rp1-network -p 7474:7474 -p 7687:7687 -d neo4j:3.4
```

2. Login to neo4j at [http://localhost:7474](http://localhost:7474) and change password from neo4j to neo4j+1.

3. rp1recengine add neo4j configurations to application.properties file.
```
# neo4j configurations
spring.data.neo4j.uri=bolt://localhost
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=neo4j+1
```

NOTE: Dependency on neo4j as previously added in [Dependency management](03_dependencies.md).

4. Add domain classes.
```java
package net.javajudd.rp1recengine.domain;

import org.neo4j.ogm.annotation.Id;
import org.neo4j.ogm.annotation.NodeEntity;

@NodeEntity
public class Song {

    @Id
    private String id;

	private String name;

	private Song() {};

	public Song(String id, String name) {
		this.id = id;
		this.name = name;
	}

	public String getId() {
		return id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

```java
package net.javajudd.rp1recengine.domain;

import java.util.HashSet;
import java.util.Set;

import org.neo4j.ogm.annotation.Id;
import org.neo4j.ogm.annotation.NodeEntity;
import org.neo4j.ogm.annotation.Relationship;

@NodeEntity
public class User {

	@Id
    private String id;

	private String username;

	private User() {};

	public User(String id, String username) {
		this.id = id;
		this.username = username;
	}

	@Relationship(type = "LIKES", direction = Relationship.UNDIRECTED)
	public Set<Song> likes;

	public void likes(Song song) {
		if (likes == null) {
			likes = new HashSet<>();
		}
		likes.add(song);
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}
}
```

5. Add repository classes.
```java
package net.javajudd.rp1recengine.repository;

import net.javajudd.rp1recengine.domain.Song;

import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface SongRepository extends CrudRepository<Song, String> {

    Song findByName(String name);
}
```

```java
package net.javajudd.rp1recengine.repository;

import java.util.List;
import net.javajudd.rp1recengine.domain.User;
import net.javajudd.rp1recengine.domain.Song;

import org.springframework.data.repository.CrudRepository;
import org.springframework.data.neo4j.annotation.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends CrudRepository<User, Long> {

    User findByUsername(String username);

    @Query("MATCH (u:User)-[:LIKES]->(commonSongs)<-[:LIKES]-(u2:User)-[:LIKES]->(recommndedSongs) WHERE u.id = {id} RETURN recommndedSongs")
    List<Song> findRecommendations(@Param("id") String id);
}
```

6. Fix broken unit tests.
```java
package net.javajudd.rp1recengine;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Rp1RecEngineApplication.class)
public class Rp1RecEngineApplicationTests {

	@Test
	public void contextLoads() {
	}

}
```


7. Update api to use Neo4j recommendations.
```java
package net.javajudd.rp1recengine.api;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import net.javajudd.rp1recengine.domain.Song;
import net.javajudd.rp1recengine.repository.UserRepository;

import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/v1")
public class ApiController {

	@Autowired
	UserRepository userRepository;

	private static final Logger logger = LoggerFactory.getLogger(ApiController.class);

	@RequestMapping("/user/{userId}/recommendation")
	public List<String> recommendations(@PathVariable("userId") String userId) {
		List<Song> recommendedSongs = userRepository.findRecommendations(userId);
		List<String> ids = recommendedSongs.stream().map(s -> s.getId()).collect(Collectors.toList());
		logger.info("User {} was recommended songs: {}", userId, ids);
		return ids;
	}
}
```

8. Run application and test [http://localhost:8080/api/v1/user/1/recommendation](http://localhost:8080/api/v1/user/1/recommendation)
```
./mvnw spring-boot:run
```


Neo4j commands:
```
// create songs
CREATE (s:Song { name: 'Thriller', id: '63a6140a-f99f-4f67-a0b2-431c1ca4c11b'})
CREATE (s:Song { name: 'Jump', id: 'ff565cd7-acf8-4dc0-9603-72d1b7ae284b'})
CREATE (s:Song { name: 'I Hate Myself for Loving Your', id: '4ac01278-4769-422f-8a0c-57c9c7c1159f'})
CREATE (s:Song { name: 'Everybody Wants to Rule the World', id: 'e135df1f-d8b3-4998-8cb4-0834585569df'})

// create users
CREATE (u:User { username: 'admin', id: 1 })
CREATE (u:User { username: 'user1', id: 2 })

// Add likes relationship
MATCH (u:User),(s:Song)
WHERE u.username = 'admin' AND s.id = 'ff565cd7-acf8-4dc0-9603-72d1b7ae284b'
CREATE (u)-[l:LIKES]->(s)
RETURN type(l)

MATCH (u:User),(s:Song)
WHERE u.username = 'admin' AND s.id = 'e135df1f-d8b3-4998-8cb4-0834585569df'
CREATE (u)-[l:LIKES]->(s)
RETURN type(l)

MATCH (u:User),(s:Song)
WHERE u.username = 'user1' AND s.id = '4ac01278-4769-422f-8a0c-57c9c7c1159f'
CREATE (u)-[l:LIKES]->(s)
RETURN type(l)

MATCH (u:User),(s:Song)
WHERE u.username = 'user1' AND s.id = '63a6140a-f99f-4f67-a0b2-431c1ca4c11b'
CREATE (u)-[l:LIKES]->(s)
RETURN type(l)

MATCH (u:User),(s:Song)
WHERE u.username = 'user1' AND s.id = 'ff565cd7-acf8-4dc0-9603-72d1b7ae284b'
CREATE (u)-[l:LIKES]->(s)
RETURN type(l)

// find songs a user likes
MATCH (u:User {username:"admin"})-[:LIKES]->(s:Song)
RETURN u, s

// find songs in common
MATCH (u:User {username:'admin'})-[:LIKES]->(s1)<-[:LIKES]-(u2:User)
RETURN s1

// recommendations
MATCH (u:User {username:'admin'})-[:LIKES]->(s1)<-[:LIKES]-(u2:User)-[:LIKES]->(s2)
RETURN s2

MATCH (u:User {id:1})-[:LIKES]->(commonSongs)<-[:LIKES]-(u2:User)-[:LIKES]->(recommndedSongs)
RETURN recommndedSongs
```
