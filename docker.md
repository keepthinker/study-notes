# docker command
## to kill a docker container process
docker kill 9f215ed0b0d3
docker pause 9f215ed0b0d3
docker unpause 9f215ed0b0d3
docker stats 9f215ed0b0d3
## make a new image from a container's changes
docker commit 70e5a5f241b8 centos-nginx-01
## create a new docker container with port mapping from 20080(host's port) -> 80(container's port) in backgroud
docker run -d -p 20080:80 centos-nginx-01  
docker ps
docker ps -a
docker run -p 8080:80 --name hello -d hello-world
docker run -it ubuntu bash
docker images
docker build -t getting-started .
docker login -u
docker login -u keepthinker
docker image ls
docker exec 6a9f550510c0 cat /data.txt
docker stop 6a9f550510c0
docker rm 6a9f550510c0
docker rm 466f31895334 83629defd935  96f729a8d7be e8a3800a262b e01445bfd6e1
docker start 82e2e71d7d79
docker attach 82e2e71d7d79
docker rename bb1a263ad46b ubuntu-bash
docker exec -i -t 0ce1dc54c68c sh

