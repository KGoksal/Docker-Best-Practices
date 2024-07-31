# DOCKER-COMPOSE BEST PRACTICES

## 1. Substitute Environment Variables in Docker Compose Files
It is best to configure those environmental variables in the machine's shell, where you will deploy the multi-container application, so there are no secret leaks. And then populate the environment variables inside the Docker Compose file by substituting as seen below.

```
mongoDB:
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_INITDB_ROOT_USERNAME}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_INITDB_ROOT_PASSWORD}
```

With the above configuration in your Docker Compose file, when you run docker-compose up, Docker Compose will look for the MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD environment variables in the shell and then substitute their values in the file.

### - How to Add Environment Variables to your Shell
Ideally, to add to a shell, you would use export VARIABLE=<variable_value>. But with that method, if the host machine reboots, those environment variables will be lost.

To add environment variables that would persist through reboots, create an environment file with:
**vi .env**
And in that file, store your environment variables like in the image below and save the file.
```
MONGO_INITDB_ROOT_USERNAME=username
MONGO_INITDB_ROOT_PASSWORD=password
```


## 2. 
