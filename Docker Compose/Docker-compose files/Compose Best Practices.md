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

## 7. Keep databases out of containers

In the micro-services / containerized crazy world we live in now, this tip is borderline heresy. You are supposed to keep everything in containers after all. However, until we have a reliable and developer environment-friendly storage solution for containers, running containers in databases means more pain than gain. Most applications use 2 to 3 different “supporting” component like databases or message buses which require persistent storage and most of us use stable versions of those: MySQL or RabbitMQ to name a few. Hosting and high availability for those components is a much older problem than container scheduling, storage persistence and high availability and as such there is much more information around on how to build a reliable MySQL Master / Slave cluster natively than running it in a container. Moreover, most databases are available as services like RDS which are scalable and reliable.

## 8. Use the same ports and load balancers

In a development environment, you don’t need to run load balancers in front of your services because you’re not going to have multiple containers for the same service. However, this is different in production. With the exception of single container services (persisted services), you are very likely to run multiple instances of the same service on each node (server). This means the load needs to be balanced across your containers using a load balancer. This is achieved by using Nginx or HAProxy in most cases. 

## 9. Always use defined volumes

Docker Compose allows you to define volumes for each service the same way they are defined with the docker run command. While this is possible it is not advisable. The best way to use volumes in Docker Compose is to define them a named volumes on top of your Compose file and use those names. This would allow your Ops team to replace the underlying storage for the volume with a SAN or NAS or other types of storage while during development you will just use a local volume mounted onto the container.


## 10. Keep your environment variables in one place

You can import them from a file like production.env or explicitly state them in your Compose file. In Compose file you can use the $ syntax to refer to process environment variables while the .env file doesn’t support that. This makes the task of finding out where the environment variables are coming from difficult and opaque. Advised on explicitly referring to all the required environment variables for a service at the point service is defined and use **the $ syntax to pull them in based on the environment. Don’t use .env files.** They make your life easier in the short term and cause hair loss in the long term.

## 11. Use strong conventions

With so many building blocks of a containerized application getting lost is very easy: git repositories, Dockerfiles, images, and services they all point to the same part of your application. While it is easy to get started with pulling the code out of your git, building it on your laptop into a Docker image and then adding that as a service to Docker compose, it will start to get confusing very quickly. 

### -  Make your containers traceable to codebase
You might use multiple git repos or a single git repo with multiple Dockerfiles or start commands for your services. While these are all ok, make sure you follow a naming convention that allows you to look at a running container and tell you exactly how that container was built, which git repo and using which Dockerfile.

### -  Make your containers traceable to git commit
This tip is about git refs and making sure you know how to trace a running container back to the git ref (git hash or tag) that was used to build it. With Docker, you now have 2 repositories to trace code through: one is your git repo and the other one is your Docker repo. This makes tracing containers back to the git ref less transparent. To avoid this use a container native CI/CD tool that is integrated with your Orchestrator (shameless plug: use Cloud 66). Short of that, use the git ref as the tag for the container running.


## 12. Make sure your services return the correct exit code

We all use logging or console output to show errors in our apps. In a containerized world, exit codes have a special place. Orchestrators (like Compose, Swarm, Kubernetes) use exit codes to tell if a service started or failed to start. The change of behavior in starting containers between Compose and Swarm means exist codes are even more important. While Docker Compose allows defining service dependencies, Docker Swarm (and Kubernetes) keep trying to start a crashing container (that exits with a code other than 0) several times. This means you need to make sure not only errors are logged but the exit codes are accurate as they will show up in the orchestrator’s logs and can give you valuable clues as to what’s going on with your application.




