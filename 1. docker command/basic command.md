If you run ```docker run ubuntu``` in the terminal, it exits immediately, because there is no process running in it.
Unlike virtual machines, containers are meant to run a specific task.

Run a container with nginx:1.14-alpine image and name it webapp
docker run -d --name webapp nginx:1.14-alpine

docker images

docker rmi \<image id>

docker run -d \<image name>

Stop a container before removing it.

Remove all of the image's container before removing the image. 