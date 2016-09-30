###### Build:

[![Build Status](https://semaphoreci.com/api/v1/garystafford/widget-service-demo/branches/master/badge.svg)](https://semaphoreci.com/garystafford/widget-service-demo)

###### Docker Hub (_status using [MicroBadger](https://microbadger.com/#/)_):

[![](https://images.microbadger.com/badges/version/garystafford/widget-service-demo.svg)](https://microbadger.com/images/garystafford/widget-service-demo "Get your own version badge on microbadger.com")  [![](https://images.microbadger.com/badges/image/garystafford/widget-service-demo.svg)](https://microbadger.com/images/garystafford/widget-service-demo "Get your own image badge on microbadger.com")

# Spring Boot Widget Microservice

#### Introduction

The µservice, the 'Widget' service, is a simple Java Spring Boot RESTful service application, backed by MongoDB and monitored with the Elastic Stack. The µservice performs basic CRUD operations on a collection of widgets, stored a MongoDB database. In the context of this post, a widget represents an inanimate object, with particular physical characteristics, such as product id, name, color, size, and current price.

Since the Widget µservice is built using Spring Boot, the final deployable form of the service is executable Spring Boot uber JAR. The JAR will contain all its dependencies and configuration. The JAR will be deployed inside the Docker image. The Docker image is considered immutable, since it contains all the necessary component and configuration to create a running Docker container, without further manipulation.

#### Technologies

-   Java
-   Spring Boot
-   Gradle
-   MongoDB
-   Spring Boot with Docker
-   The Elastic Stack (ELK and Filebeat)
-   DockerHub
-   Semaphore CI
-   HashiCorp Packer

#### MongoDB

Import sample data to MongoDB locally

```bash
# from your project root
mongoimport \
  --host localhost:27017 \
  --db widgets \
  --collection widget \
  --type json --jsonArray \
  --file src/main/resources/data/data.json
```

Common MongoDB commands

```bash
mongo # use mongo shell
> show dbs
> use widgets
> db.widget.find()
> db.dropDatabase()
```

#### Build Service Locally

```bash
./gradlew clean build \
  && java -jar build/libs/widget-service-0.2.0.jar
```

#### Test Service Locally

Create a new widget

```bash
curl -i -X POST -H "Content-Type:application/json" -d '{
  "product_id": "N212QZOD9B",
  "name": "Pentwist",
  "color": "Gray",
  "size": "Huge",
  "price": 75,
  "inventory": 5,
  "preview": "https://s3.amazonaws.com/widgets-microservice-demo/N212QZOD9B.png"
}' http://localhost:8030/widgets
```

Create another new widget

```bash
curl -i -X POST -H "Content-Type:application/json" -d '{
  "product_id": "N43WV5234S",
  "name": "Zapster",
  "color": "Green",
  "size": "Tiny",
  "price": 5,
  "inventory": 4,
  "preview": "https://s3.amazonaws.com/widgets-microservice-demo/N43WV5234S.png"
}' http://localhost:8030/widgets
```

Get all widgets

```bash
curl -i -X GET http://localhost:8030/widgets | prettyjson
```

#### Building Docker Images with Spring Boot with Docker

Change the `group` key in `build.gradle` to you DockerHub repository name, such as

```text
group = '<your_dockerhub_repo_name>'
```

Login to your Docker Hub account from command line

```bash
docker login
```

Build the Docker Image containing the Spring Boot service JAR

```bash
./gradlew clean build buildDocker
```

If `push = true` was set in the `buildDocker` method of the `build.gradle`, the images
is automatically pushed to your DockerHub account.

If you chose to set `push = false` within the `buildDocker` method of the `build.gradle`,
then use the following type of command to push the new Docker Image to DockerHub, after it is built.

```bash
docker push <your_dockerhub_repo_name>/widget-service:latest
```

Create and run a Docker container

```bash
docker run -e "SPRING_PROFILES_ACTIVE=production" -p 8030:8030 -t garystafford/widget-service
```

Import sample data to MongoDB running in the Docker container

```bash
# from your project root
mongoimport \
  --host localhost:27018 \
  --db widgets \
  --collection widget \
  --type json --jsonArray \
  --file src/main/resources/data/data.json
```

Alternately, use images built with Spring Boot with Docker and Gradle from `Dockerfile` files

```bash
docker-compose -f docker-compose.yml -p widget up -d elk; docker logs --follow elk # ^C to exit
docker-compose -f docker-compose.yml -p widget up -d mongodb; docker logs --follow mongodb
docker-compose -f docker-compose.yml -p widget up -d widget; docker logs --follow widget
```

#### References

-   <https://github.com/Transmode/gradle-docker>
-   <https://github.com/bmuschko/gradle-docker-plugin>
