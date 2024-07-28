# Docker Best Practices

## Best Practice Rules
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








