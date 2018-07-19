# Backing Services

Treat backing services as attached resources

1. Start neo4j database.
```
docker run --name rp1-neo4j --network rp1-network -p 7474:7474 -p 7687:7687 -d neo4j:3.4
```

2. Login to neo4j at [http://localhost:7474](http://localhost:7474) and change password to neo4j+1.

3. rp1recengine add neo4j configurations to application.properties file.
```
# neo4j configurations
spring.data.neo4j.uri=bolt://localhost
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=neo4j+1
```

NOTE: Dependency on neo4j as previously added in [Dependency management](03_dependencies.md).

4. Add domain classes.
```
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

```
package net.javajudd.rp1recengine.domain;

import java.util.Collections;
import java.util.HashSet;
import java.util.Optional;
import java.util.Set;
import java.util.stream.Collectors;

import org.neo4j.ogm.annotation.GeneratedValue;
import org.neo4j.ogm.annotation.Id;
import org.neo4j.ogm.annotation.NodeEntity;
import org.neo4j.ogm.annotation.Relationship;

@NodeEntity
public class User {

    @Id 
    private Long id;

	private String username;

	private User() {};

	public User(Long id, String username) {
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
}
```

5. Add repository classes.
```
package net.javajudd.rp1recengine.repository;

import net.javajudd.rp1recengine.domain.Song;

import org.springframework.data.repository.CrudRepository;

public interface SongRepository extends CrudRepository<Song, String> {

    Song findByName(String name);
}
```

```
package net.javajudd.rp1recengine.repository;

import java.util.List;
import net.javajudd.rp1recengine.domain.User;
import net.javajudd.rp1recengine.domain.Song;

import org.springframework.data.repository.CrudRepository;
import org.springframework.data.neo4j.annotation.Query;
import org.springframework.data.repository.query.Param;

public interface UserRepository extends CrudRepository<User, Long> {

    User findByUsername(String username);

    @Query("MATCH (u:User)-[:LIKES]->(commonSongs)<-[:LIKES]-(u2:User)-[:LIKES]->(recommndedSongs) WHERE u.id = {id} RETURN recommndedSongs")
    List<Song> findRecommendations(@Param("id") Long id);
}
```

6. Update api to use Neo4j recommendations.
```
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
public class ApiController {

	@Autowired
	UserRepository userRepository;

	private static final Logger logger = LoggerFactory.getLogger(ApiController.class);

	@RequestMapping("/user/{userId}/recommendation")
	public List<String> recommendations(@PathVariable("userId") Long userId) {
		List<Song> recommendedSongs = userRepository.findRecommendations(userId);
		List<String> ids = recommendedSongs.stream().map(s -> s.getId()).collect(Collectors.toList());
		logger.info("User {} was recommended songs: {}", userId, ids);
		return ids;
	}
}
```

7. Run application and test [http://localhost:8080/user/1/recommendation](http://localhost:8080/user/2/recommendation)
```
./mvnw spring-boot:run
```