# Photo App API Spring Boot + Spring Cloud

This Backend application has been build with Spring Boot, Spring Cloud, RabbitMQ, Elastic Search, Kibana, and Logstash.

## RabbitMQ

```zsh
docker run -d --name rabbit-name-management -p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 rabbitmq:3-management-alpine
```

## Config Server

```zsh
docker build --tag=config-server --force-rm=true ./PhotoAppApiConfigServer
```

```zsh
docker run -d -p 8012:8012 -e "spring.rabbitmq.host={RABBITMQ-URL}" config-server
```

## Eureka Discovery Server

```zsh
docker build --tag=eureka-server --force-rm=true ./PhotoAppDiscoveryService
```

```zsh
docker run -d -p 8761:8761 -e "spring.config.import=optional:configserver:{CONFIG_SERVER_HOST}" eureka-server
```

## API Gateway

```zsh
docker build --tag=api-gateway --force-rm=true ./PhotoAppApiGateway
```

```zsh
docker run -d -p 8082:8082 -e "spring.config.import=optional:configserver:{CONFIG_SERVER_HOST}" api-gateway
```

## Elastic Search

```zsh
docker network create --driver bridge api-network
```

```zsh
docker run -d -v esdata1:/usr/share/elasticseach/data --name elasticsearch -p 9200:9200 -p 9300:9300 -e “discovery.type=single-node” --network api-network elasticsearch:7.16.3
```

## Kibana

```zsh
docker run -d -p 5601:5601 --network api-network kibana:7.16.3
```

## Albums Microservice

```zsh
docker build --tag=albums-microservice --force-rm=true ./PhotoAppApiAlbums
```

```zsh
docker run -d -e "eureka.client.service-url.defaultZone=http://test:test@{EUREKA_HOST}/eureka" -e "logging.file=/api-logs/albums-ws.logs" -v {DIRECTORY}/api-logs:/api-logs albums-microservice
```

## Albums Logstash

```zsh
docker build --tag=albums-logstash --force-rm=true ./ELK/albums-logstash
```

```zsh
docker run -d --name albums-microservice-logstash -v {DIRECTORY}/api-logs:/api-logs albums-logstash
```

## MySQL

```zsh
docker run -d -p 3306:3306 --name mysql-docker-container -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=photo_app -e MYSQL_USER=user -e MYSQL_PASSWORD=password -v /var/lib/mysql:/var/lib/mysql mysql:8
```

## Users Microservice

```zsh
docker build --tag=users-microservice --force-rm=true ./PhotoAppApiUsers
```

```zsh
docker run -d -e "spring.config.import=optional:configserver:{CONFIG_SERVER_HOST}" -e "logging.file=/api-logs/users-ws.logs" -v {DIRECTORY}/api-logs:/api-logs users-microservice
```

## Users Logstash

```zsh
docker build --tag=users-logstash --force-rm=true ./ELK/users-logstash
```

```zsh
docker run -d --name users-microservice-logstash -v {DIRECTORY}:/api-logs users-logstash
```
