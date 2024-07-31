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





