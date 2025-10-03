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
```
version: '3.0'
services:
  redis:
  image: redis:alpine

  clickcounter:
  image: kodekloud/click-counter
  ports:
  - 8085:5000
```
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

ADD  abc.tar.gz  /app

- Using Add u can download from a url also

ADD  https://abc.com  /app

### RUN

* The RUN instruction is used in Dockerfiles to execute commands that build and configure the Docker image.
* These commands are executed during the image build process, and each RUN instruction creates a new layer in the Docker image.
* For example, if you create an image that requires specific software or libraries installed, you would use RUN to execute the necessary installation commands.
** eg :-- RUN apt update && apt -y install apache2 **

### CMD
* The CMD instruction specifies the default command to run when a container is started from the Docker image.
* **CMD can be overridden by supplying command-line arguments to docker run.**
  eg:-
  * For example, by default, you might want a web server to start, but users could override this to run a shell instead:
  * CMD ["apache2ctl", "-DFOREGROUND"]
 
  * Users can start the container with docker run -it <image> /bin/bash to get a Bash shell instead of starting Apache.
 
### ENTRYPOINT

* Any arguments supplied to the docker run command are appended to the ENTRYPOINT command.
* Use ENTRYPOINT when you need your container to always run the same base command, and you want to allow users to append additional commands at the end.
* ENTRYPOINT can be overridden on the docker run command line by supplying the --entrypoint flag.
* For example, suppose you are packaging a custom script that requires arguments (e.g., “my_script extra_args”). In that case, you can use ENTRYPOINT to always run the script process (“my_script”) and then allow the image users to specify the “extra_args” on the docker run command line
* ENTRYPOINT ["my_script"]

### Combining CMD and ENTRYPOINT
* The CMD instruction can be used to provide default arguments to an ENTRYPOINT if it is specified in the exec form.
* This setup allows the entry point to be the main executable and CMD to specify additional arguments that can be overridden by the user.

**ENTRYPOINT ["python", "/app/my_script.py"]***
**CMD ["--default-arg"]**

Running docker run myimage --user-arg executes **python /app/my_script.py --user-arg**.
