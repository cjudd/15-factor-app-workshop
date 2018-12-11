# Administrative Processes

Run admin/management tasks as one-off processes.

1. Add new class to batch load Neo4j.
```java
package net.javajudd.rp1recengine;

import net.javajudd.rp1recengine.domain.Song;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

import net.javajudd.rp1recengine.domain.User;
import net.javajudd.rp1recengine.repository.SongRepository;
import net.javajudd.rp1recengine.repository.UserRepository;
import org.springframework.jdbc.support.rowset.SqlRowSet;

import java.util.Optional;

@SpringBootApplication
public class PopulateNeo4j {

    private static final Logger log = LoggerFactory.getLogger(PopulateNeo4j.class);

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    UserRepository userRepository;

    @Autowired
    SongRepository songRepository;

    public static void main(String args[]) {
        System.setProperty("spring.main.web-application-type", "NONE");
        ConfigurableApplicationContext context = SpringApplication.run(PopulateNeo4j.class, args);
        PopulateNeo4j bean = context.getBean(PopulateNeo4j.class);
        bean.run();
        System.exit(0);
    }

    public void run() {

        log.info("Delete data");
        userRepository.deleteAll();
        songRepository.deleteAll();

        log.info("Populate data");

        jdbcTemplate.query(
            "select * from song", new Object[] {},
            (rs, rowNum) -> new Song(rs.getString("guid"), rs.getString("name"))
        ).forEach(song -> songRepository.save(song));

         jdbcTemplate.query(
                 "select * from user", new Object[] {},
                 (rs, rowNum) -> new User(rs.getString("id"), rs.getString("username"))
         ).forEach(user -> userRepository.save(user));

        SqlRowSet srs = jdbcTemplate.queryForRowSet("select u.username as username, s.guid as guid " +
                "from user u " +
                "join user_song us on us.user_id = u.id " +
                "join song s on us.song_id = s.id");
        while (srs.next()) {
            User user = userRepository.findByUsername(srs.getString("username"));
            Optional song = songRepository.findById(srs.getString("guid"));
            song.ifPresent(s -> {
                user.likes((Song)s);
                userRepository.save(user);
            });
        }

        log.info("Done");
    }
}
```

2. Run class.
```
./mvnw -Dmainclass=net.javajudd.rp1recengine.PopulateNeo4j spring-boot:run
```

3. Package and create Docker image.
```
./mvnw package
```

4. Run from container.
```
docker run -it --rm --network rp1-network --entrypoint "/usr/bin/java"  -eSPRING_DATA_NEO4J_URI=bolt://rp1-neo4j -eSPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 javajudd/rp1-rec-engine:0.0.1-SNAPSHOT  -cp /usr/share/rp1/rp1-rec-engine.jar -Dloader.main=net.javajudd.rp1recengine.PopulateNeo4j org.springframework.boot.loader.PropertiesLauncher
```

5. Validate Neo4j was populated.