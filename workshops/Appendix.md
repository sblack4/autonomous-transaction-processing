# Appendix 

Here are some handy docker commands you may need while doing <a href="./LabGuide800BuildingMicroservicesOnATP.md" target="_blank">LabGuide800BuildingMicroservicesOnATP.md</a>

### List all images in your local docker repository

```
$ docker images -a
```

### List running docker containers

```
$ docker ps -a
```

### Clean up dangling in-memory containers

```
$ docker system prune
```

### Delete all docker images

```
$ docker rmi -f$(docker images -q)
```

### Build a docker container

```
$ docker build –t <tagname> -f <Dockerfile_name>
```

### Map ports on localhost (say port 80)to container and run it

```
$ docker run –i –p 80:80 –t <image_name> 
```
