# Description

Sample of using microservices in docker containers. Used libraries: OAuth, Security, OpenFeign, Zuul, Lombok.
All services are logging in ELK stack.

```sh
@Component
public class AuthRequestInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate requestTemplate) {
        Authentication _auth = SecurityContextHolder.getContext().getAuthentication();
        if (_auth != null) {
            if (_auth.getDetails() instanceof  OAuth2AuthenticationDetails) {
                String _token = ((OAuth2AuthenticationDetails) _auth.getDetails()).getTokenValue();
                requestTemplate.header("Authorization", String.format("Bearer %s", _token));
            }
        }
    }
}
```

# Installing ELK

Before running your application you should start Elastic Stack tools on your machine.
The best way (simply way) to do that is through Docker.
Firstly, let's create the network:
```sh
$ docker network create app_net
```

Secondly, let's run ElasticSearch:
```sh
$ docker run -d --name elasticsearch --net app_net -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:latest
```

In next step, let's run Logbash:
```sh
ocker run -d --name logstash --net app_net -p 5000:5000 -v ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf docker.elastic.co/logstash/logstash:latest
```

```
input {
  tcp {
    port => 5000
    codec => json
  }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "micro-%{appname}"
  }
}
```

Finally, let's run Kibana:
```sh
1
$ docker run -d --name kibana --net app_net -e "ELASTICSEARCH_URL=http://elasticsearch:9200" -p 5601:5601 docker.elastic.co/kibana/kibana:latest
```

# Alternative way of installing ELK

[https://github.com/deviantony/docker-elk#how-to-configure-elasticsearch]

# Running db

```sh
docker-compose -f ./docker-compose-db.yml up -d --build
```
or

```sh
sh ./build-db.sh
```

# Running the application

```sh
mvn clean package
docker-compose -f ./docker-compose.yml up -d --build
```

or

```sh
sh ./build-dev.sh
```

# Application Endpoints

| Service | Port | Endpoint |
| ------ | ------ | ------ |
| Eureka | 8761 | http://localhost:8761/<resource>
| AuthService | 8811 | http://localhost:8811/oauth/<resource>
| Service1 | 8812 | http://localhost:8812/svc1/<resource>
| Service2 | 8813 | http://localhost:8813/svc2/<resource>
| GatewayService | 8810 | http://localhost:8810/api/<resource>




