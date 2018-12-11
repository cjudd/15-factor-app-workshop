# API First

1. Use a browser to navigate to [Spring Initializer](http://start.spring.io/).

2. Generate a RP1 Recommendation Engine project containing the following:
* build - Maven
* language - Java
* spring boot version - newest
* group - net.javajudd
* artifact - rp1recengine
* name - rp1-rec-engine
* description - RP1 Music Recommendation Engine
* package - net.javajudd.rp1recengine
* dependencies - web

or 

```
curl https://start.spring.io/starter.zip -d dependencies=web \
-d name=rp1-rec-engine -d artifactId=rp1recengine -d groupId=net.javajudd -d packageName=net.javajudd.rp1recengine -d description="RP1 Music Recommendation Engine" \
-d language=java -d javaVersion=1.8 -d packaging=jar -d type=maven-project -d baseDir=rp1recengine -o rp1recengine.zip
```

3. Unzip rp1recengine.zip.
```
unzip rp1recengine.zip
cd rp1recengine
```

4. Version control rp1recengine (See [Codebase](01_codebase.md)).

```
git init
git add .
git commit -m "Created a Spring Boot 2.X app using start.spring.io containing Web dependency."
```

5. Add stubbed out net.javajudd.rp1recengine.api.ApiController class.

```
package net.javajudd.rp1recengine.api;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;
import java.util.Random;

@RestController
@RequestMapping("/api/v1")
public class ApiController {

    private Random random = new Random();
    private static final int SONG_COUNT = 11;

    private static final Logger logger = LoggerFactory.getLogger(ApiController.class);

    @RequestMapping("/user/{userId}/recommendation")
    public List<Integer> recommendations(@PathVariable("userId") int userId) {
        List<Integer> recommendedSongs = Arrays.asList(random(), random(), random());
        logger.info("User {} was recommended songs: {}", userId, recommendedSongs);
        return recommendedSongs;
    }

    private int random() {
        return random.nextInt(SONG_COUNT);
    }
}
```

6. Run api.

```
./mvnw spring-boot:run
```

7. Test api by navigating to [http://localhost:8080/api/v1/user/1/recommendation](http://localhost:8080/api/v1/user/1/recommendation)

8. Shut down api.