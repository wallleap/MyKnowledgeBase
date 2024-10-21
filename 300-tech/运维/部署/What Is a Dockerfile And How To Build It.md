---
title: What Is a Dockerfile And How To Build It
date: 2024-08-30T09:48:59+08:00
updated: 2024-08-30T09:50:00+08:00
dg-publish: false
source: https://spacelift.io/blog/dockerfile
---

Docker is a platform for building and running containerized applications. Containers package your source code, dependencies, and runtime environment into reusable units, which can be deployed anywhere a container runtime is available. This could be a Docker installation on your laptop or a [Kubernetes cluster](https://spacelift.io/blog/kubernetes-tutorial) that provides your production infrastructure.

Wherever you launch your containers, you need a container image to run. Images define the initial state of container filesystems. They’re created using a [Dockerfile](https://docs.docker.com/engine/reference/builder) – a list of instructions that Docker processes to assemble your image.

In this article, we’ll walk through what Dockerfiles are, how to create one, and some best practices you should follow.

## What Is a Dockerfile?

**Dockerfiles** are text files that list instructions for the Docker daemon to follow when building a container image. When you execute the [`docker build` command](https://docs.docker.com/engine/reference/commandline/build), the lines in your Dockerfile are processed sequentially to assemble your image.

The basic Dockerfile syntax looks as follows:

```bash
# Comment
INSTRUCTION arguments
```

Instructions are keywords that apply a specific action to your image, such as copying a file from your working directory, or executing a command within the image.

By convention, instructions are usually written in uppercase. This [is not a requirement](https://docs.docker.com/engine/reference/builder) of the Dockerfile parser, but it makes it easier to see which lines contain a new instruction. You can spread arguments across multiple lines with the backslash operator:

```bash
RUN apt-get install \
curl \
	wget
```

Lines starting with a `#` character are interpreted as comments. They’ll be ignored by the parser, so you can use them to document your Dockerfile.

### Key Dockerfile Instructions

Docker supports over [15 different Dockerfile instructions](https://docs.docker.com/engine/reference/builder) for adding content to your image and setting configuration parameters. Here are some of the most common ones you’ll use.

#### 1. `FROM`

```bash
FROM ubuntu:20.04
```

`FROM` is usually the first line in your Dockerfile. It refers to an existing image which becomes the base for your build. All subsequent instructions apply on top of the referenced image’s filesystem.

#### 2. `COPY`

```bash
COPY main.js /app/main.js
```

`COPY` adds files and folders to your image’s filesystem. It copies between your Docker host and the work-in-progress image. Containers using the image will include all the files you’ve copied in.

The instruction’s first argument references the source path on your host. The second argument sets the destination path inside the image. It’s possible to directly copy from *another* Docker image using the `--from` flag:

```bash
# Copies the path /usr/bin/composer from the composer:2 image
COPY --from=composer:2 /usr/bin/composer composer
```

3. `ADD`

```bash
ADD http://example.com/archive.tar /archive-content
```

`ADD` works similarly to `COPY` but additionally supports remote file URLs and automatic archive extraction. Archives will be extracted into the destination path in your container. Decompression of `gzip`, `bzip2`, and `xz` formats is supported.

Although `ADD` can simplify some image authorship tasks, its use [is discouraged](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) because its behavior masks important details. Accidentally using `ADD` instead of `COPY` can be confusing because archive files will be extracted into your container instead of being copied as-is.

4. `RUN`

```bash
RUN apt-get update && apt-get install -y nodejs
```

`RUN` runs a command inside the image you’re building. It creates a new [image layer](https://docs.docker.com/storage/storagedriver/#images-and-layers) on top of the previous one; this layer will contain the filesystem changes that the command applies. `RUN` instructions are most commonly used to install and configure packages that your image requires.

#### 5. `ENV`

```bash
ENV PATH=$PATH:/app/bin
```

The `ENV` instruction is used to set environment variables that will be available within your containers. Its argument looks similar to a variable assignment in your shell: specify the name of the variable and the value to assign, separated by an equals character.

## Example: Creating and Using a Dockerfile[](https://spacelift.io/blog/dockerfile#example-creating-and-using-a-dockerfile)

Ready to create a Docker image for your application? Here’s how to get started with a simple Dockerfile.

First, create a new directory for your project. Copy the following code and save it as `main.js`:

```bash
const {v4: uuid} = require("uuid");

console.log(`Hello World`);
console.log(`Your ID is ${uuid()}`);
```

Use npm to add the `uuid` package to your project:

```bash
$ npm install uuid
```

Next, copy the following set of Docker instructions, then save them to `Dockerfile` in your working directory:

```bash
FROM node:16
WORKDIR /app

COPY package.json .
COPY package-lock.json .
RUN npm install

COPY main.js .

ENTRYPOINT ["node"]
CMD ["main.js"]

```

Let’s unpack this Dockerfile line-by-line:

- `FROM node:16` – [The official Node.js image](https://hub.docker.com/_/node) is selected as the base image. All the other statements apply on top of `node:16`.
- `WORKDIR /app` – The working directory is changed to `/app`. Subsequent statements which use relative paths, such as the `COPY` instructions immediately afterwards, will be resolved to `/app` inside the container.
- `COPY package.json` – The next two lines copy in your `package.json` and `package-lock.json` files from your host’s working directory. The destination path is `.`, meaning they’re deposited into the in-container working directory with their original names.
- `RUN npm install` – This instruction executes `npm install` inside the container’s filesystem, fetching your project’s dependencies.
- `COPY main.js .` – Your app’s source code is added to the container. It happens after the `RUN` instruction because your code will usually change more frequently than your dependencies. This order of operations allows more [optimal usage](https://docs.docker.com/build/cache) of Docker’s build cache.
- `ENTRYPOINT ["node"]` – The image’s [entrypoint](https://docs.docker.com/engine/reference/builder/#entrypoint) is set so the `node` binary is launched automatically when you create new containers with your image.
- `CMD ["main.js"]` – This instruction supplies arguments for the image’s entrypoint. In this example, it results in Node.js running your application’s code.

### Building your Dockerfile

Now you can use `docker build` to build an image from your Dockerfile. Run the following command in your terminal:

```bash
$ docker build -t demo-image:latest .
```

Wait while Docker builds your image. The sequence of instructions will be displayed in your terminal.

### Docker Build Options

The `docker build` command takes the path to the *build context* as an argument. The build context defines which paths you can reference within your Dockerfile. Paths outside the build context will be invisible to Dockerfile instructions such as `COPY`. It’s most common to set the build context to `.`, as in this example, to refer to your working directory.

Docker automatically looks for instructions in your working directory’s `Dockerfile`, but you can reference a different Dockerfile with the `-f` flag:

```bash
$ docker build -f dockerfiles/app.dockerfile -t demo-image:latest .
```

The `-t` flag sets the tag which will be assigned to your image after the build completes. You can repeat the flag if you need to add multiple tags.

### Using Your Image

With your image built, start a container to see your code execute:

```bash
$ docker run demo-image:latest
Hello World!
Your ID is 606c3a30-e408-4c77-b631-a504559e14a5
```

The image’s filesystem has been populated with the Node.js runtime, your npm dependencies, and your source code, based on the instructions in your Dockerfile.

💡 You might also like:

- [Common Infrastructure Challenges and How to Solve Them](https://spacelift.io/blog/infrastructure-problems-that-spacelift-solves)
- [16 DevOps Best Practices to Follow](https://spacelift.io/blog/devops-best-practices)
- [Top Most Useful CI/CD Tools for DevOps](https://spacelift.io/blog/ci-cd-tools)

## Dockerfile Best Practices[](https://spacelift.io/blog/dockerfile#dockerfile-best-practices)

Writing a Dockerfile for your application is usually a relatively simple task, but there are some common gotchas to watch out for. Here are 10 best practices you should follow to maximize usability, performance and security.

### 1. Don’t use `latest` for your base images

Using an image such as `node:latest` in your `FROM` instructions is risky [because it can expose you to](https://spacelift.io/blog/docker-tutorial) unexpected breaking changes. Most image authors immediately switch latest to new major versions as soon as they’re released. Rebuilding your image could silently select a different version, causing a broken build or malfunctioning container software.

Selecting a specific tag such as `node:16` is safer because it’s more predictable. Only use `latest` when there’s no alternative available.

### 2. Only use trusted base images

Similarly, it’s important to choose trusted base images to protect yourself from backdoors and security issues. The content of the image referenced by your `FROM` instruction is included in your image; compromised base images could contain malware that runs inside your containers. Where possible, try to use Docker Hub images that are marked as [official or submitted by a verified publisher](https://hub.docker.com/search?q=&image_filter=official%2Cstore).

### 3. Use `HEALTHCHECK` to enable container health checks

Health checks tell Docker and administrators when your containers enter a failed state. Orchestrators such as Docker Swarm and Kubernetes can use this information to automatically restart problematic containers.

Enable health checks for your containers by adding a [`HEALTHCHECK` instruction](https://docs.docker.com/engine/reference/builder/#healthcheck) to your Dockerfile. It sets a command Docker will run inside the container to check whether it’s still healthy:

```bash
HEALTHCHECK --timeout=3s CMD curl -f http://localhost || exit 1
```

The healthiness of your containers is displayed when you run the docker ps command to list them:

```bash
$ docker ps
CONTAINER ID   IMAGE                	COMMAND              CREATED    	STATUS
335889ed4698   demo-image:latest      "httpd-foreground"   2 hours ago	Up 2 hours (healthy)
```

### 4. Set your `ENTRYPOINT` and `CMD` correctly

`ENTRYPOINT` and `CMD` are [closely related instructions](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact). `ENTRYPOINT` sets the process to run when a container starts, while `CMD` provides default arguments for that process. You can easily override `CMD` by setting a custom argument when you start containers with `docker run`.

In the example Dockerfile created above, `ENTRYPOINT ["node"]` and `CMD ["main.js"]` result in `node main.js` executing when the container is started with `docker run demo-image:latest`.

If you ran `docker run demo-image:latest app.js`, then Docker would call `node app.js` instead.

Read more about [the differences between Docker ENTRYPOINT and CMD](https://spacelift.io/blog/docker-entrypoint-vs-cmd).

### 5. Don’t hardcode secrets into images

Dockerfiles shouldn’t contain any [hardcoded secrets](https://spacelift.io/blog/docker-secrets) such as passwords and API keys. Values set in your Dockerfile apply to all containers using the image. Anyone with access to the image can inspect your secrets.

Set environment variables [when individual containers start](https://docs.docker.com/engine/reference/commandline/run/#env) instead of providing defaults in your Dockerfile. This prevents accidental security breaches.

### 6. Label your images for better organization

Teams with many different images often struggle to organize them all. You can set arbitrary metadata on your images using the Dockerfile `LABEL` instruction. This provides a convenient way to attach relevant information that’s specific to your project or application. By convention, labels are commonly set using reverse DNS syntax:

```bash
LABEL com.example.project=api
LABEL com.example.team=backend
```

Container management tools usually display image labels and let you filter to different values.

### 7. Set a non-root user for your images

Docker defaults to running container processes as `root`. This is problematic because `root` in the container is the same as `root` on your host. A malicious process which escapes the container’s isolation could run arbitrary commands on your Docker host.

You can mitigate this risk by including the `USER` instruction in your Dockerfile. This sets the user and group which your container will run as. It’s good practice to assign a non-root user in all of your Dockerfiles:

```bash
# set the user
USER demo-app

# set the user with a UID
USER 1000

# set the user and group
USER demo-app:demo-group
```

### 8. Use `.dockerignore` to prevent long build times

The [build context](https://docs.docker.com/build/building/context) is the set of paths that the `docker build` command has access to. Images are often built using your working directory as the build context via docker build ., but this can cause redundant files and directories to be included.

Paths which aren’t used by your Dockerfile, or which will be recreated inside the container by other instructions, should be removed from the build context to improve performance. This will save time when Docker copies the build context at the start of the build process.

Add a `.dockerignore` file to your working directory to exclude specific files and directories. The syntax is similar to [`.gitignore`](https://git-scm.com/docs/gitignore):

```bash
.env
.local-settings
node_modules/
```

### 9. Keep your images small

Docker images can become excessively large. This slows down build times and increases transfer costs when you move your images between registries.

Try to reduce the sizes of your images by only installing the minimum set of packages required for your software to function. It also helps to use compact base images when possible, such as [Alpine Linux](https://hub.docker.com/_/alpine) (5 MB), instead of larger distributions like Ubuntu (28 MB).

### 10. Lint your Dockerfile and scan images for vulnerabilities

Dockerfiles can contain errors that break your build, cause unexpected behavior, or violate best practices. Use a linter such as [Hadolint](https://github.com/hadolint/hadolint) to check your Dockerfile for problems before you build.

Hadolint is easily run using its own Docker image:

```bash
$ docker run --rm -i hadolint/hadolint < Dockerfile
```

The results will be displayed in your terminal.

You should also scan built images for vulnerabilities. Container scanners such as [Trivy](https://github.com/aquasecurity/trivy) can detect outdated packages and known CVEs inside your image’s filesystem. Running a scan before you deploy helps prevent exploitable containers from reaching production environments.

Check out also [Docker security best practices](https://spacelift.io/blog/docker-security).

## Key Points[](https://spacelift.io/blog/dockerfile#key-points)

Docker is [one of the most popular](https://www.docker.com/blog/key-insights-from-stack-overflows-2022-developer-survey) developer technologies. It simplifies modern software delivery tasks by packaging code and dependencies into containers that function identically across multiple environments.

To use Docker, you must first write a Dockerfile for your application. This contains instructions that Docker uses to create your container image. Dockerfiles can copy files from your local build context, run commands within your image’s filesystem, and set metadata such as labels that help to organize different images.

The Dockerfiles you write aren’t limited to Docker. The Dockerfile filename is commonly used by convention, but more generic alternatives, such as [Podman’s favored Containerfile](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users) are gaining momentum. Dockerfile instructions are defined by the [Open Container Initiative (OCI)](https://opencontainers.org/) image specification and produce images that will work with any OCI-compatible container runtime.

Packaging software as a container makes it more portable, allowing you to eliminate discrepancies between environments. You can use the container on your laptop, in production, and within your CI/CD infrastructure. Take a look at how [Spacelift](https://spacelift.io/) uses Docker containers to run CI jobs. You have the possibility of bringing your own Docker image and [using it as a runner](https://docs.spacelift.io/integrations/docker) to speed up the deployments that leverage third party tools. Spacelift’s official runner image can be found [here](https://github.com/spacelift-io/runner-terraform).
