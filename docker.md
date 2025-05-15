docker

docker login

docker tag todoapp-docker:latest shettyp/todoapp-docker:latest
docker images
docker push shettyp/todoapp-docker:latest

docker pull shettyp/todoapp-docker:latest

docker run -dp 3000:80 shettyp/todoapp-docker:latest

docker exec -it containername sh
or
docker exec -it 15c6a2bc582f sh

docker logs containername
or
docker logs 15c6a2bc582f

docker inspect

docker image rm image-id

docker run --name redis -d redis:alpine

docker run --name clickcounter -p 8085:5000 -d --link redis:redis kodekloud/click-counter

version: '3.0'
services:
  redis:
  image: redis:alpine

  clickcounter:
  image: kodekloud/click-counter
  ports:
  - 8085:5000

docker-compose up -d

********************************************************************************

docker build --no-cache -t py-project .   --->  no cacheing will be used to build the image and it will dowload the dependencies everytime 

docker build -t ekart -f docker/Dockerfile .    --->   if the docker file location is elsewhere 

docker build -t boardgame:$(git rev-parse --short HEAD) .   --->  if u want to change the tag name to git commit number for every build

docker build -t boardgame:release-$(date +%Y-%m-%d) .   --->  if u want to change the tag name to current date for every build

docker run --rm --memory="512m" --cpus=1.5" boardgame:latest  --->  set up ram & cpu for container. Also using the rm command the container will not be visible in the "docker ps -a" output after it is stopped

docker run -u 1001:1001 boardgame:latest   --->  run docker with different user 

docker run --read-only -v /tmp --tmpfs /tmp -d -p 8080:8080 boardgame:latest   ---> this creates read only container with rw in /tmp only

docker run -d --restart=on-failure:3 alpine sh -c "sleep 1 && exit 1" ---> container will restart if it fails with exit code 1

HEALTHCHECK CMD curl --fail http://localhost:8080/health || exit 1 ---> add this to docker file for health status

docker logs -f --tail 50 container-id ---> tail docker logs  

docker exec -it container-id sh -c "ps aux|grep java" --> get details from inside the container

docker run -it --pids-limit 100 --memory=256m --read-only alpine sh ---> run a container with maximum 100 process

docker run -v -d $(pwd)/logs:var/log/nginx nginx ---> map the current OS directory to nginx logs directory in container so no need to go inside container to access logs

docker rmi $(docker images -f "dangling=true" -q) ---> delete dangling images i.e images that do not have a name or tag 

docker image ls -f "since=xyz:v1" ---> lists the images created after xyz:v1

docker rename oldname newname  ---> rename a container

docker update --restart=always container-name ---> update container policy to restart always when failed

docker pause container-name && docker unpause container-name ---> to pause and unpause container

docker diff container-name ---> to see changes made in container

docker save image-name > filename.tar ---> export docker image to a tar file

docker load < filename.tar ---> to import the docker image from a tar file

docker cp container-name:/location/filename.txt ./filename.txt ---> copy file from container to local machine

docker stats --no-stream ---> get utilization (cpu,memory, etc) of all the containers running 

### Copy vs ADD
Copy and Add are similar, except that Add has some advantages over copy

- Using Add u can extract an image also

RUN  abc.tar.gz  /app

- Using Add u can download from a url also

RUN  https://abc.com  /app
