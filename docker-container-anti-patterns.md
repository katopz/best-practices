# Docker Container Anti Patterns
> Credit : http://blog.arungupta.me/docker-container-anti-patterns/

## Data or logs in containers
Containers are ideal for stateless applications and are meant to be ephemeral. This means no data or logs should be stored in the container otherwise they’ll be lost when the container terminates. Instead use [volume mapping](http://blog.couchbase.com/2016/october/persisting-couchbase-data-across-container-restarts) to persist them outside the containers. [ELK stack](http://blog.arungupta.me/getting-started-elk-stack-wildfly/) could be used to store and process logs. If managed volumes are used during early testing process, then remove them using `-v` switch with the `docker rm` command.

## IP addresses of container
Each container is assigned an IP address. Multiple containers communicate with each other to create an application, for example an application deployed on an application server will need to talk with a database. Existing containers are terminated and new containers are started all the time. Relying upon the IP address of the container will require constantly updating the application configuration. This will make the application fragile. Instead create services. This will provide a logical name that can be referred independent of the growing and shrinking number of containers. And it also provides a basic load balancing as well.

## Run a single process in a container
A `Dockerfile` has use one `CMD` and `ENTRYPOINT`. Often, CMD will use a script that will perform some configuration of the image and then start the container. Don’t try to start multiple processes using that script. Its important to follow _separation of concerns_ pattern when creating Docker images. This will make managing your containers, collecting logs, updating each individual process that much harder. You may consider breaking up application into multiple containers and manage them independently.

## Don’t use `docker exec`
The `docker exec` command starts a new command in a running container. This is useful for attaching a shell using the docker exec -it {cid} bash. But other than that the container is already running the process that its supposed to be running.

## Keep your image lean
Create a new directory and include Dockerfile and other relevant files in that directory. Also consider using .dockerignore to remove any logs, source code, logs etc before creating the image. Make sure to remove any downloaded artifacts after they are unzipped.

## Create image from a running container
A new image can be created using the `docker commit` command. This is useful when any changes in the container have been made. But images created using this are non-reproducible. Instead make changes in the Dockerfile, terminate existing containers and start a new container with the updated image.

## Security credentials in Docker image
Do not store security credentials in the Dockerfile. They are in clear text and checked into a repository. This makes them completely vulnerable. Use `-e` to specify passwords as runtime environment variable. Alternatively `--env-file` can be used to read environment variables from a file. Another approach is to used `CMD` or `ENTRYPOINT` to specify a script. This script will pull the credentials from a third party and then configure your application.

## `latest` tag
Starting with an image like `couchbase` is tempting. If no tags are specified then a container is started using the image `couchbase:latest`.  This image may not actually be latest and instead refer to an older version. Taking an application into production requires a fully controller environment with exact version of the image. Read this [Docker: The latest confusion](http://container-solutions.com/docker-latest-confusion/) post by fellow Docker Captain [@adrianmouat](http://twitter.com/adrianmouat).  Make sure to always use tag when running a container. For example, use `couchbase:enterprise-4.5.1` instead of just `couchbase`.

## Impedance mismatch
Don’t use different images, or even different tags in dev, test, staging and production environment. The image that is the “source of truth” should be created once and pushed to a repo. That image should be used for different environments going forward. In some cases, you may consider running your unit tests on the WAR file as part of maven build and then create the image. But any system integration testing should be done on the image that will be pushed in production.

## Publishing ports
Don’t use `-P` to publish all the exposed ports. This will allow you to run multiple containers and publish their exposed ports. But this also means that all the ports will be published. Instead use `-p` to publish specific ports.

## Root user
 Don’t run containers as root user. The host and the container share the same kernel. If the container is compromised, a root user can do more damage to the underlying hosts. Use `RUN groupadd -r couchbase && useradd -r -g couchbase couchbase` to create a group and a user in it. Use the `USER ` instruction to switch to that user. Each `USER` creates a new layer in the image. Avoid switching the user back and forth to reduce the number of layers. Thanks to [@Aleksandar_78](https://twitter.com/Aleksandar_78/status/79299790148823449) for this tip!

## Dependency between containers
Often applications rely upon containers to be started in a certain order. For example, a database container must be up before an application can connect to it. The application should be resilient to such changes as the containers may be terminated or started at any time. In this case, have the application container wait for the database connection to succeed before proceeding further. Do not use wait-for scripts in Dockerfile for the containers to startup in a specific order. Particularly waiting for a certain number of seconds for a particular container to start is very fragile. Thanks to [@ratnopam](https://twitter.com/ratnopam/status/79311589400412979) for this tip!