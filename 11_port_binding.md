# Port Binding

Export services via port binding.

1. Rebuild container.
```
./mvnw package
```

2. Start the recommendation api as a Docker container.
```
docker run --name rp1-api -d --network rp1-network -p9090:8080 -eSPRING_DATA_NEO4J_URI=bolt://rp1-neo4j -eSPRING_DATASOURCE_URL=jdbc:mysql://rp1-music-db:3306/rp1 javajudd/rp1-rec-engine:0.0.1-SNAPSHOT
```

3. Test api by navigating to [http://localhost:9090/api/v1/user/1/recommendation](http://localhost:9090/api/v1/user/1/recommendation).

