
## Init a node image in docker

`docker image build -t node-app .`: `.` means from current directory.

`docker container run -p 80:3000 node-app`: mapping the host `localhost:80` to container's 3000.

`docker image tag node-app chocoluffy/node-helloworld`: tag the image with proper name. 

`docker push chocoluffy/node-helloworld`: push the image to the remote DockerHub repository.


## Notes

### docker completion in zsh

[tutorial](https://docs.docker.com/machine/completion/#zsh)

### container
`docker container run --publish 80:80 nginx`: publish means to map the traffic of host port to the client port.

`docker container run --publish 80:80 --detach --name myhost nginx`

`docker container ls`: shows running container.

`docker container ls -a`: shows history.

`docker container stop 6`

`docker container logs myhost`

`docker container top myhost`: show pid

`docker container rm 6` or `docker container rm -f 6`: use unique id to remove container.

* find image from cache, if not found, then fetch from docker hub.
* then assign ip and port

> containers are just processes.

`docker container run --name mongo -d mongo`: call it mongo and run it on the background and using mongo image.

`docker container inspect`

`docker container stats`: real time report of container.

`docker container run -it --name proxy nginx bash`, after the image name, it follows the “command name”, that will run at the start of starting an image. “-it” means going ssh into the container and shell interactively.

`docker container run -it --name ubuntu ubuntu`, will pull an image of ubuntu up and enter the shell.

> always try typing `--help` to see the detailed of all available options.

`docker container start -ai proxy`, will start an existed stop image.

`docker container ls -a`: will show all image cache, including not running ones.

OR use `docker container exec -it --name proxy nginx bash`, different with `run` is that `exec` will not overwrite the original command, but instead it adds the additional commands besides.

`docker pull <image name>`

`docker image ls`: show all images.

### docker network

virtual network, so that the container in the same virtual network can communicate with each other freely. And can only expose to the public if specify by `-p port number`. And the default network is called “bridge”(the network NATed, behind the host IP); `docker network ls`: can list all the existing network.

`docker network create new-app`

`docker container run -d --net new-app --net-alias search elasticsearch:2`, and this command can be run in twice.

* the default bridge network, cannot do the `--link` job, otherwise to create a new virtual network with driver. And later the `docker-compose` will ease the process so much(every time it will create custom network).
* container should not rely on IP address for inter-communication. DNS for friendly name is built-in if using custom network!! Thus, always using customer network

> image: just the binary of application; and the host provides kernel.

### image layer

when create an image, we create an layer.  We use Dockerfile to add some files on top of that. And on top of that, maybe we have env files.
`docker image history` will show all layer of an image. 

* images are made up of file system changes, and meta-data.
* each layer is uniquely identified and only stored once on the host. so that it saves time on host and transfer time on push\pull.

### image tag

tag: git tag. a pointer to a specific commit. 

`docker image tag nginx chocoluffy/nginx`

`docker image push chocoluffy/nginx`

`docker login`

### Dockerfile

each command is each layer. 

`RUN ` will follow bash commands, which use `&&` to concat so that they all belong to the same layer. 

`FROM ` will gives the minimum distribution. It will pull from the docker hub and cached. and then execute line by line.

`docker image build -t customniginx .`: build the image on the current directory.

thus the ordering in the docker file matters! say if we “copy the source code” at the first line, overtime I changes my source code, it will rebuild the whole things again!

> Let the thing changing the most at the bottom of Dockerfile.

```python
# this same shows how we can extend/change an existing official image from Docker Hub

FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn

WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is prefered to using 'RUN cd /some/path'

COPY index.html index.html

# I don't have to specify EXPOSE or CMD because they're in my FROM

```

`docker container run -p 80:80 --rm nginx`: `--rm` meaning we don’t need the image, it will delete it right after we stop. 

> Dockerfile -> image -> container

`docker image build -t nginx-with-html .`: will build the image.
so that we can run the container with the latest image.

`docker container run -p 80:80 —rm  nginx-with-html`

> The author open source his best practice on node with docker: [GitHub - BretFisher/node-docker-good-defaults: sample node app for Docker examples](https://github.com/BretFisher/node-docker-good-defaults)

### persistent data

* container is immutable.
* persistent data: Volumes, and Bind Mounts.

even if we remove the container, the data is saved. it will save a specific address of data on the host. 

`docker volume ls`: check all existing volumes. 
`docker container run -d --name mysql -v mysql-db:/var/lib/mysql mysql`: naming the volume and its path.

bind mounting: maps a host file to a container file directory. and cannot be specified in Dockerfile. 

`docker container run -d --name nginx -p 80:80 -v $(pwd):/usr/share/nginx/html nginx`: will start a nginx server and map the current directory to one specify, so that the current html file will overwrite the default one. [bind-mount’s path starts with `/`]

> once we map such bind-mount, whenever we create files from the host file directory, it will reflect on the container side!

`docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve`

### docker compose

  YAML file that describes solution for:
* containers
* networks
* volumes

docker-compose for local docker automation only?

in production, use Swarm.
`docker-compose up`
`docker-compose down`

so that onboard the new person will be `git clone` and `docker-compose up`

```python
version: '3'

services:
  proxy:
    image: nginx:1.13 # this will use the latest version of 1.11.x
    ports:
      - '80:80' # expose 80 on host and sent to 80 in container
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
  web:
    image: httpd  # this will use httpd:latest

# so that in this case, we want the nginx not as serving up static files; but used as a reverse proxy to direct traffic for the server behind.

# network is not requried. 
```















 









