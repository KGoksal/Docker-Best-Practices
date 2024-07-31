# Docker Best Practices
## A. BEST PRACTICE RULES
### 1. Use an official and verified Docker image as a base image, whenever available.

Instead of taking a base operating system image (like ubuntu) and installing node.js, npm and whatever other tools you need for your application, **use the official node image for your application.**
### 2. Use specific Docker image versions
So instead of a random latest image tag, you want to fixate the version and just like you deploy your own application with a specific version you want to use the official image with a specific version. 
-  **the more specific the better** Transparency to know exactly what version of the base image you're using

### 3. Use Small-Sized Official Images
When choosing a Node.js image, you will see there are actually multiple official images. Not only with different version numbers, but also with different operating system distributions. So pay attention to these:

-  **1) Image Size**
  In contrast having **smaller images** means you need **less storage** space in image repository as well as on a deployment server and of course you can transfer the images **faster when pulling or pushing** them from the repository.
-  **2) Security Issue**
-  In comparison by using smaller images with leaner OS distributions, which only bundle the necessary system tools and libraries, you're also **minimizing the attack surface and making sure that you build more secure images.**
-  = So the best practice here would be to select an image with a **specific version based on a leaner OS distribution like alpine** for example: Alpine has everything you need to start your application in a container, but is much more lightweight. And for most of the images that you look on a Docker Hub, you will see a version tag with alpine distribution inside.
### 4. OPTIMIZE CACHING FOR IMAGE LAYERS WHEN BUILDING AN IMAGE 
When you rebuild your image, if your Dockerfile hasn't changed, Docker will just use the cached layers to build the image.
**Advantages of cached image layers:**
- ✅ **Faster image building**
- ✅ **Faster pulling and pushing of new image versions:** Only the newly added layers will be downloaded, the rest are already locally cached by Docker.
- ✅ **Optimize the Caching**
- Order your commands in the Dockerfile from the least to the most frequently changing commands to take advantage of caching and this way optimize how fast the image gets built. 🚀

### 5. Use .dockerignore file
Create this .dockerignore file and list all the files and folders that we want to be ignored and when building the image, Docker will look at the contents and ignore anything specified inside. It leads to reduce image size We don't need the auto-generated folders, like: 
- targets,
- build folder,
- readme file etc.

### 6. Make use of Multi-Stage Builds
Let's say there are some contents (**like development, testing tools and libraries**) in your project that you need for building the image - so during the
build process - but **you DON'T NEED them in the final image itself to run the application**. If you keep these artifacts in your final image even though they're absolutely unnecessary for running the application, it will again result in an increased image size and increased attack surface. 
- = separate the build stage from the runtime stage.

- The multi-stage builds feature allows you to use **multiple temporary images during the build process**, but **keep only the latest image as the final artifact**:
So these previous steps will be discarded.
_- Separation of Build Tools and Dependencies from what's needed for runtime_
_- Less dependencies and reduced image size_


### 7. USE THE LEAST PRIVILEGED USER 
-> By default, when a Dockerfile does not specify a user, it uses a root user. But in reality there is mostly no reason to run containers with root privileges. This basically introduces a security issue, because when container starts on the host it, will potentially have root access on the Docker host. So running an application inside the container with a root user will make it easier for an attacker to escalate privileges on the host and basically get hold of the underlying host and its processes, not only the container itself. Especially if the application inside the container is vulnerable to exploitation.

- To avoid this, the best practice is to simply create a dedicated user and a dedicated group in the Docker image to run the application and also run the application inside the container with that user:
- You can use a directive called USER with the username and then start the application conveniently.
- **Tip:** Some images already have a generic user bundled in, which you can use. So you don't have to create a new one. For example the node.js image already bundles a generic user called node, which you can simply use to run the application inside the container. 

### 8. Scan your Images for Security Vulnerabilities
Once you build the image to scan it for security vulnerabilities using the **docker scan command**. In the background Docker actually uses a service called snyk to do the vulnerability scanning of the images. The scan uses a database of vulnerabilities, which gets constantly updated. You can see:
    1) the type of vulnerability,
    2) a URL for more information
    3) but also what's very useful and interesting you see which version of the relevant library actually fixes that vulnerability. So you can update your libraries to get rid of these issues.
       
In addition to scanning your images manually with docker scan command on a CLI, you can also configure Docker Hub to scan the images automatically, when they get pushed to the repository. And of course you can integrate this check in your CI/CD pipeline when building your Docker images.


****************************************************************************************

## B. TIPS
The **--rm** flag in Docker is used to automatically **remove the container when it exits.** This is useful for running short-lived containers where you don't need to keep the container around after it has completed its task, thus helping to keep your system clean from unused containers.

```
Copy code
docker run --rm my-image
```
### PRUNE:
  - **docker image prune** to clean up just "dangling" images
  - **docker system prune** will clean up everything you're not currently using
  - The big one is usually **docker image prune -a** which will remove all images you're not using.
  - Use **docker system df** to see space usage.
  - **Linux/macOS bash**, sh, zsh, and Windows Docker Toolbox Quickstart Terminal use: **$(pwd)** which means "print working directory".
  - **For PowerShell** use: **${pwd}**
  - **For cmd.exe** "Command Prompt use: **%cd%**
  - When running postgres, you'll need to either set a password, or tell it to allow any connection you need to either set a password with the environment variable:

POSTGRES_PASSWORD=mypasswd

Or tell it to ignore passwords with the environment variable:

POSTGRES_HOST_AUTH_METHOD=trust

-> Use the command **ps aux** in each container to see a list of processes and usernames. The process needs a matching user ID or group ID to access the files in question.
Note: If ps doesn't work in your container, you may need to install it. In debian-based images with apt, you can add it with apt-get update && apt-get install procps



****************************************************************************************

## C. BEST PRACTICE SAMPLES

