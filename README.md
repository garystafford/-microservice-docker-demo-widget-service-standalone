###### Build:  
![https://travis-ci.org/garystafford/microservice-docker-demo-widget.svg?branch=master](https://travis-ci.org/garystafford/microservice-docker-demo-widget.svg?branch=master)

###### Docker Hub:  
[![](https://images.microbadger.com/badges/version/garystafford/microservice-docker-demo-widget.svg)](http://microbadger.com/images/garystafford/microservice-docker-demo-widget "Get your own version badge on microbadger.com")  [![](https://images.microbadger.com/badges/image/garystafford/microservice-docker-demo-widget.svg)](http://microbadger.com/images/garystafford/microservice-docker-demo-widget "Get your own image badge on microbadger.com")

# Spring Boot Widget Microservice

#### Introduction
One of a set of Java Spring Boot services, for an upcoming post on scaling microservices with the latest Spring and Docker features. A Docker Image, containing the service, is be build locally and pushed to DockerHub, using Spring Boot with Docker. Alternately, the post will demonstrate building the Docker Image, using a continuous integration pipeline with GitHub, Travis CI and DockerHub automated build feature.

Widgets are inanimate objects that users purchase with points. Widgets have particular physical characteristics, such as product id, name, color, size, and current price. An inventory of widgets is stored in the `widgets` MongoDB database.

#### Technologies
* Java
* Spring Boot
* Gradle
* MongoDB
* Spring Cloud Config Server (migrating to Consul)
* Spring Cloud Netflix Eureka
* Spring Boot with Docker

#### MongoDB
Import sample data to MongoDB locally
```bash
# set your project root
PROJECT_ROOT='/Users/gstaffo/Documents/projects/widget-docker-demo'
mongoimport --host localhost:27017 --db widgets --collection widget \
  --type json --jsonArray \
  --file ${PROJECT_ROOT}/widget-service/src/main/resources/data/data.json
```

Common MongoDB commands
```bash
mongo # use mongo shell
> show dbs
> use widgets
> db.widget.find()
> db.dropDatabase()
```

#### Build Service
Build and start service locally
```bash
./gradlew clean build && \
  java -jar -Dspring.profiles.active=local \
  build/libs/widget-service-0.1.0.jar
```

#### Test Service
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

#### Building Images with Spring Boot
Change the `group` key in `build.gradle` to you DockerHub repository name, such as
```text
group = '<your_dockerhub_repo_name>'
```

Login to your Docker Hub account from command line
```bash
docker login
```

Build the Docker Image containing service jar
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

Import sample data to MongoDB running in container
```bash
# set your project root
PROJECT_ROOT='/Users/gstaffo/Documents/projects/widget-docker-demo'
mongoimport --host localhost:27018 --db widgets --collection widget \
  --type json --jsonArray \
  --file ${PROJECT_ROOT}/widget-service/src/main/resources/data/data.json
```

#### References
* [https://github.com/Transmode/gradle-docker](https://github.com/Transmode/gradle-docker)
