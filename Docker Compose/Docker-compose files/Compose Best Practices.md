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
Then in the root directory of your production machine, run $ ls -la, which should show you a .profile

Open up the profile file with $ vi .profile and at the end of the file, add this configuration:

```
set -o allexport; source /<path_to_the_directory_of_.env_file>/.env; set +o allexport
```

And to ensure the configuration takes effect, log out of your shell session, log back in, and then run:

```
printenv
```


## 2. If Possible, Avoid Multiple Compose Files for Different Environments

As a rule, you should always try to keep only a single Docker Compose file for all environments. But in cases where you might want to use nodemon to monitor changes in your node application during development. Or perhaps you would use a managed MongoDB database in production but run MongoDB locally in a container for development.

For these cases, you can use the docker-compose.override.yml file. As its name implies, the override file will contain configuration overrides for existing or entirely new services in your docker-compose.yaml file.

To run your project with the override file, you still run the default docker-compose up command. Then Docker Compose will automatically merge the docker-compose.yml and docker-compose.override.yml files into one. Suppose you defined a service in both files; Docker Compose merges the configurations using the rules described in Adding and overriding configuration.

You can add the override file to your .gitignore file, so it won't be there when you push code to production. After doing this, if you still see the need to create more Compose files, then go ahead but keep in mind the complexity.


## 3. Use YAML Templates to Avoid Repetition

When a service has options that will repeat in other services, you can create a template from the initial service to reuse in the other services instead of continuously repeating yourself.
The following illustrates Docker Compose YAML templating:

```
version: '3.9'
services:
  web: &service_default
    image: .
    init: true
    restart: always 
  backend:
    <<: *service_default # inheriting the service defaults definitions
    image: <image_name>
    env_file: .env
    environment:
      XDEBUG_CONFIG: "remote_host=${DOCKER_HOST_NAME_OR_IP}"
```

In the above YAML configuration, the first service defines restart: always, which will restart the container automatically if it crashes. Instead of adding restart: always and other recurring configs you might have to all your services, you can replicate them with <<: *service_default.


# Production
## 1. Leverage the Docker Restart Policy

Occasionally, you'll face a scenario when a service fails to start. A common reason is that another service on your host machine has changed, and Docker Compose uses the old environment variables. To ensure this doesn't happen, set the restart behavior to restart: always and configure your services with **update_config: true**. This will refresh the environment variables for each run. However, if your app relies on other services (MySQL, Redis, etc.) outside of Docker Compose, then you should take extra precautions. Make sure they are configured correctly.

## 2. Correct Cleanup Order of Docker Images

You need to clean up the order of your images during production. Do not use docker rm -f as it may destroy useful images. Always run **docker rm -f --remove-orphans**. If you're working in the dev stage, this is not an issue because Docker Compose builds images only once, then exposes them. Thus there's no need to worry about removing old images. However, in production, Docker loops through all images when the container stops and restarts.

Consequently, there's no way for you to be sure that an image wasn't destroyed, even when docker-compose down is called. If a container is stopped and restarted, then the exposed images can change, and you can't be sure they're still in use. Using docker rm -f to delete containers is a mistake. Docker Compose reuses port bindings, so an old service is still available, even though its container was destroyed.

Since you cannot tell which containers might be potentially in use, you must delete all of them using the --remove-orphans flag. If a container is restarted by Docker Compose (or something else) and it reuses the same port, the new image will have the same image ID as the old one.

Notice we've added the --remove-orphans flag because that ensures Docker Compose only deletes containers and images that are no longer in use, regardless of whether we or a running container uses them. This is crucial if you have services restarting.

## 3. Setting Your Containers' CPU and Memory Limits

You can configure Docker to limit the CPU and memory of your containers by passing arguments into the docker-compose.yml file before starting your container. For example, the following command will start a web service with one CPU:
```
web:
    deploy:
      resources:
        limits:
          cpus: "1"
```

If you set a specific number of CPUs in the multi_cpu key, it will only be used when available. If you fail to set the limit, the service will use the maximum resources it requires.

If you set a specific number of CPUs in the multi_cpu key, it will only be used when available. If you fail to set the limit, the service will use the maximum resources it requires.

Tip: If you want to run multiple containers with different memory limits on the same machine, ensure that all your containers have different memory limits. This is because each container views how much memory it needs.

**Note:** You can use this technique for multiple services if you'd like. Docker Compose will automatically get the values from the env file for each container when it starts up.

Consequently, you need to understand the resource requirements of your service. This will prevent you from wasting resources and minimize production costs.

## 4. Keep your compose file the same across dev and production
 While Docker suggests having different Docker Compose files in development and production, we have seen many cases that this has caused issues. By keeping your development and production files separate, at the best, you end up reapplying all modifications in your development Compose file to your production one. This is a manual and error-prone process. In many cases, some changes are missed and your app breaks just as it goes to production. This approach also ignores other environments than development and production, like Staging or QA. Any new environment will add to this complexity. As a rule, you should always try to keep only a single Docker Compose file for all environments. Some of the tips below will elaborate more on some of the ways of doing so.

## 5. Keep your compose file version controlled and next to the app

This means any changes in your application should be reflected in your Docker Compose file. Changes to the port of a service or a new service added to the application should be made in your Docker Compose file as well. That's why the best place for your Docker Compose file is next to your app and version controlled in lockstep with your application.

## 6. Keep images EXPLICITLY VERSIONE 

This might seem obvious but keeping images explicitly versioned in your Compose file saves a lot of hassle later on. By default, Docker uses latest as the tag for an image when no tag is specified. While this behavior works fine in development and is very convenient (since you don’t need to change the Compose file every time you make a change to a service), it makes tracking of changes difficult and running in production indeterministic. It is best to explicitly specify the version of each service in the Compose file. You can use **the ${ENV} format** to get this version from an external source like your git ref or another parameter to avoid changing your Compose file every time.



