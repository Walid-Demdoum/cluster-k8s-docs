## Common docker commands
### Pull image from dockerhub

```bash
docker pull <image>
```
### Start/Stop container
```bash
docker start/stop <container-name/id>
```
### Enter bash mode inside container
```bash
docker -u <user> -it <container> /bin/bash
```

### Save image as tar to be copied/ Load image from tar
```bash
docker save -o <path/to/filename.tar> <image>
docker load -i <path/to/filename.tar>
```

### Run Odoo container from command line
#### It is always prefered to do from a docker-compose file
```bash
docker run -d -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:15
docker run -p <host port>:<container port> --name <container_name> --link <db_container>:db -t <image_name>
```

### Show docker system wide
```bash
docker system df -v
```
### Docker run conainer with desired ports
```bash
docker run -dp "port host::gettingaccessed" : "port to publish::inside container" <image name>
```

### Image creating 
#### We assume that that your registry is already set up
```bash
docker build -t <imagename:tag> .
docker tag <image:tag> <registry>/<image:tag>
docker push <registry>/<imagename:tag>
```