
## Install test container
    docker build -t gitsecret-test .
    docker run -it --name gitsecret -d gitsecret-test:latest
    docker exec -it gitsecret bash

## Remove test container
    docker stop gitsecret
    docker container rm gitsecret
    docker image rm gitsecret-test

## 
