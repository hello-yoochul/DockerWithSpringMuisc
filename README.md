## Prerequisites

[spring-music](https://github.com/cloudfoundry-samples/spring-music) 클론 먼저 하기 

```
git clone https://github.com/cloudfoundry-samples/spring-music.git
cd spring-music
./gradlew clean assemble
java -jar build/libs/spring-music.jar # http://localhost:8080/ 들어가서, 실행 확인 (jar 파일 이름 다를 수도 있으니, path 들어가서 확인 필요)
```

build 하기

```
./gradlew clean assemble
```

앱 실행 해보기 (http://localhost:8080/ 들어가서 확인)

```
java -jar build/libs/spring-music.jar 
```

주의, jar 파일 이름 다를 수도 있으니, path 들어가서 확인 필요

```
╭─yoochulkim@YOOCHULs-MacBook-Pro ~/IdeaProjects/DOCKER
╰─$ tree -L 4
./DockerWithSpringMuisc
├── build
│   ├── libs
│   │   └── spring-music-1.0.jar
```

# 주의 사항

- docker 디렉토리에 [1](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/1) -> [2](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/2) -> [3](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/3) 폴더 순차적으로 최상위에 디렉토리에 복붙할 것이며, 2를 진행할때 1에서 가져온 파일을 다 지워줘야 한다, 2 -> 3 으로 갈때도 마찬가지.

# 1 Dockerfile

목표: 기본적인 도커 이미지 만들어보기

우선, [폴더 1](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/1) 에 있는 Dockerfile 가져온다.

도커 이미지 생성 

```
sudo docker build -t my-springmusic .
```

이미지 확인

```
docker images | grep my-springmusic
```

도커 이미지를 컨테이너로 실행하기

```
docker run -it -p 8080:8080 my-springmusic
```

Dockerfile에서 명시한 jar 파일 COPY 된것 확인

```
╭─yoochulkim@YOOCHULs-MBP ~/IdeaProjects/DOCKER/DockerWithSpringMuisc ‹master●›
╰─$ docker exec -it 07d6cda57539 sh                                                       
/ # ls
app.jar  bin      dev      etc      home     lib      media    mnt      opt      proc     root     run      sbin     srv      sys      tmp      usr      var
```

구동중인 컨테이너 확인

```
docker ps
```

웹사이트 들어가서 spring-music 사이트 확인

```
http://localhost:8080/
```



# 2 Docker Compose

목표: Docker Compose 를 이용해 이미지 만들고 컨테이너로 실행해보기.

폴더 1 에서 가져온것들을 루트 디렉토리에서 모두 삭제한다 (즉, Dockerfile만).

폴더 2 에 있는 모든 파일 (Dockerfile, docker-compose.yml) 을 가져온다.

클린 환경 만들기

```
docker kill $(docker ps -q) # 실행 중인 모든 컨테이너 종료
docker container prune # 멈춰있는 모든 컨테이너 삭제 
docker image prune # 모든 이미지 삭제
```

도커 이미지 만들기

```
sudo docker-compose -f docker-compose.yml build
docker images # 빌드된 이미지 확인
```

실행 

```
sudo docker-compose up (백그라운드로 돌릴시 -d 옵션 추가)
```

웹사이트 들어가서 spring-music 사이트 확인

```
http://localhost:8080/
```

종료

```
docker-compose down
```

# 3 Spring 앱과 DB 연동하기 

목표: 위에 까지는 spring profile 을 명시를 지정해주지 않아 (Spring Music [README.md](https://github.com/cloudfoundry-samples/spring-music#running-the-application-locally) 보기), in-memory로 사용됐다 ([resource 에 있는 album 데이터 로드되서 사용되었음](https://github.com/cloudfoundry-samples/spring-music/blob/master/src/main/resources/albums.json)). Spring Muisic 앱이 MongoDB 를 사용하도록 연동해보자. 여러 도커 컨테이너 간에 네트워크 형성도 할 것이다.

[폴더 2](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/2) 에서 가져온것들을 루트 디렉토리에서 모두 삭제한다.

[폴더 3](https://github.com/hello-yoochul/DockerWithSpringMuisc/tree/master/docker/3) 에 있는 모든 폴더와 파일을 가져온다. 아래와 같이 디렉토리 구조가 나오도록 한다.

```
─yoochulkim@YOOCHULs-MacBook-Pro ~/IdeaProjects/DOCKER
╰─$ tree -L 3 ./
./
└── DockerWithSpringMuisc
    ├── Dockerfile
    ├── LICENSE
    ├── README.md
    ├── build
    │   ├── bootJarMainClassName
    │   ├── classes
    │   ├── generated
    │   ├── libs
    │   ├── resources
    │   └── tmp
    ├── build.gradle
    ├── docker
    │   ├── 1
    │   ├── 2
    │   └── 3
    ├── docker-compose-mongodb.yml
    ├── docker-compose-springapp.yml
    ├── gradle
    │   └── wrapper
    ├── gradle.properties
    ├── gradlew
    ├── gradlew.bat
    ├── manifest.yml
    ├── mongo-entrypoint
    │   ├── init-users.sh
    │   └── seed-music.js
    ├── my-mongoclient
    │   └── Dockerfile
    ├── project.toml
    ├── settings.gradle
    └── src
        ├── main
        └── test
```

클린 환경 만들기

```
docker kill $(docker ps -q) # 실행 중인 모든 컨테이너 종료
docker container prune # 멈춰있는 모든 컨테이너 삭제 
docker image prune # 모든 이미지 삭제
```

DB 먼저 띄우기

```
docker compose -f docker-compose-mongodb.yml build
docker compose -f docker-compose-mongodb.yml up
```

네트워크 확인 (`bridge`, `host`, and `none` 는 도커가 생성하는 기본 네트워크 - [참고](https://stackoverflow.com/questions/41083328/what-is-the-use-of-docker-host-and-none-networks))

```
╭─yoochulkim@YOOCHULs-MBP ~/IdeaProjects/DOCKER/DockerWithSpringMuisc ‹master●›
╰─$ docker network ls                                                                     
NETWORK ID     NAME                              DRIVER    SCOPE
33007520faf1   bridge                            bridge    local
e996fcc064ef   dockerwithspringmuisc_mongo_net   bridge    local
d016e430b97f   host                              host      local
62b195a5e4e5   none                              null      local
```

client와 mongodb 모두 같은 네트워크에 있음 확인

```
╭─yoochulkim@YOOCHULs-MBP ~/IdeaProjects/DOCKER/DockerWithSpringMuisc ‹master●›
╰─$ docker inspect -f '{{.HostConfig.NetworkMode}}' my-mongoclient
dockerwithspringmuisc_mongo_net

╭─yoochulkim@YOOCHULs-MBP ~/IdeaProjects/DOCKER/DockerWithSpringMuisc ‹master●›
╰─$ docker inspect -f '{{.HostConfig.NetworkMode}}' my-mongodb                            
dockerwithspringmuisc_mongo_net
```

스프링 앱 띄우기

```
docker compose -f docker-compose-springapp.yml build
docker compose -f docker-compose-springapp.yml up
```

스프링 띄어져 있는 리눅스 안에 설정된 앱 관련 환경 변수 확인 

```
╭─yoochulkim@YOOCHULs-MBP ~/IdeaProjects/DOCKER/DockerWithSpringMuisc ‹master●›
╰─$ docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}'  dockerwithspringmuisc-my-springmusic-1
SPRING_PROFILES_ACTIVE=mongodb,remote
spring.data.mongodb.uri=mongodb://admin:admin@my-mongodb:27017/test
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin
LANG=C.UTF-8
JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
JAVA_VERSION=8u212
JAVA_ALPINE_VERSION=8.212.04-r0
```

사이트 들어가보기 (앨범 추가도 해보기)

```
http://localhost:8080/
```

데이터 확인 

```
╭─yoochulkim@YOOCHULs-MBP ~
╰─$ docker exec -it my-mongoclient sh                                                      # mongo $MONGO_SERVER/$MONGO_INITDB_DATABASE -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD
MongoDB shell version v4.0.5
connecting to: mongodb://my-mongodb:27017/test?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("16286359-43aa-4a99-aeac-d99209711fbb") }
MongoDB server version: 4.2.3
WARNING: shell and server versions do not match
> show collections
album
test
> db.album.find().pretty()
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d6697"),
	"title" : "Nevermind",
	"artist" : "Nirvana",
	"releaseYear" : "1991",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d6698"),
	"title" : "Pet Sounds",
	"artist" : "The Beach Boys",
	"releaseYear" : "1966",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d6699"),
	"title" : "What's Going On",
	"artist" : "Marvin Gaye",
	"releaseYear" : "1971",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669a"),
	"title" : "Are You Experienced?",
	"artist" : "Jimi Hendrix Experience",
	"releaseYear" : "1967",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669b"),
	"title" : "The Joshua Tree",
	"artist" : "U2",
	"releaseYear" : "1987",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669c"),
	"title" : "Abbey Road",
	"artist" : "The Beatles",
	"releaseYear" : "1969",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669d"),
	"title" : "Rumours",
	"artist" : "Fleetwood Mac",
	"releaseYear" : "1977",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669e"),
	"title" : "Sun Sessions",
	"artist" : "Elvis Presley",
	"releaseYear" : "1976",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d669f"),
	"title" : "Thriller",
	"artist" : "Michael Jackson",
	"releaseYear" : "1982",
	"genre" : "Pop",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a0"),
	"title" : "Exile on Main Street",
	"artist" : "The Rolling Stones",
	"releaseYear" : "1972",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a1"),
	"title" : "Born to Run",
	"artist" : "Bruce Springsteen",
	"releaseYear" : "1975",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a2"),
	"title" : "London Calling",
	"artist" : "The Clash",
	"releaseYear" : "1980",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a3"),
	"title" : "Hotel California",
	"artist" : "The Eagles",
	"releaseYear" : "1976",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a4"),
	"title" : "Led Zeppelin",
	"artist" : "Led Zeppelin",
	"releaseYear" : "1969",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a5"),
	"title" : "IV",
	"artist" : "Led Zeppelin",
	"releaseYear" : "1971",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a6"),
	"title" : "Synchronicity",
	"artist" : "Police",
	"releaseYear" : "1983",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a7"),
	"title" : "Achtung Baby",
	"artist" : "U2",
	"releaseYear" : "1991",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a8"),
	"title" : "Let it Bleed",
	"artist" : "The Rolling Stones",
	"releaseYear" : "1969",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66a9"),
	"title" : "Rubber Soul",
	"artist" : "The Beatles",
	"releaseYear" : "1965",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
{
	"_id" : ObjectId("64a9bce7c9c0060cb09d66aa"),
	"title" : "The Ramones",
	"artist" : "The Ramones",
	"releaseYear" : "1976",
	"genre" : "Rock",
	"trackCount" : 0,
	"_class" : "org.cloudfoundry.samples.music.domain.Album"
}
Type "it" for more
```

# 도커 유용한 명령어

- `docker inspect <<컨테이너 이름>>`
- `docker-compose -f <<문제 컨테이너 파일명>> down --remove-orphans`

# 참고 자료

- https://fabianlee.org/2018/05/24/docker-running-a-spring-boot-based-app-in-a-docker-container/
- https://fabianlee.org/2018/05/24/docker-running-a-spring-boot-based-app-using-docker-compose/
- https://fabianlee.org/2018/05/20/docker-using-docker-compose-to-link-a-mongodb-server-and-client/
- https://fabianlee.org/2018/05/26/docker-using-docker-compose-and-networking-to-link-a-spring-boot-app-to-an-external-service-dependency/

