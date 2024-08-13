
docker build -t adminnik/go-web-app:v2 . ----> Build a dockerimage file for which you have wrote a Dockerimage file. 
Here we have tagged the file as well. Dont forget to use . or whatever path the dockerfile is located in.

docker run -p 8080:8080 adminnik/go-web-app:v2 - After dockerimage is build successfully, to run container 

docker images - gives you the all docker images that you have build

docker push adminnik/go-web-app:v2  - Push image to docker hub


docker build --platform linux/amd64 -t adminnik/go-web-app:3 . - build a docker image which is compatible with `amd64` architecture nodes in your EKS cluster.

docker image inspect adminnik/go-web-app:v3 | grep Architecture