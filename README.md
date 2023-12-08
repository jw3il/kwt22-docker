# KWT22 Docker Workshop

This workshop teaches you the absolute basics of docker within 60 minutes.
Its main purpose is to motivate you to use docker in your own projects.

This readme complements the slides provided in the `slides/` directory ([PDF link](slides/KWT22_Docker_in_60_minutes.pdf)).

We will barely scratch the surface of docker's capabilities and focus on selected parts from the [get started guide](https://docs.docker.com/get-started/) in docker's documentation. 
More details and information about advanced functionality can be found on the [official docker website](https://docs.docker.com/).

## Step 0: Install docker (preparation)

You can install docker on Linux, Mac and Windows. 
Have a look at the [official install guides](https://docs.docker.com/get-docker/) for more information.

*This step should be done before the workshop starts, especially since the Windows installation with WSL 2 can take a while.*

## Step 1: Hello World (5 min)

We will start exploring docker basics with a simple hello world example.

### Task

1. Run the `hello-world` docker image

    ```
    docker run hello-world
    ```

    What happens when you execute this command? Read the response that is printed in your terminal. The newly created container will directly exit.

2. Verify that you now have a local copy of the `hello-world` image...

    ```
    docker image ls
    ```
3. ..and a corresponding container with a random name that has already exited.
    ```
    docker container ls --all
    ```
4. We won't need these anymore, so let's delete them.
    ```
    docker container prune
    docker image rm hello-world
    ```

### [Optional Bonus Task]

If you are very fast and there is time left, you can follow the output of hello-world and interactively run bash on an ubuntu container with `-it`

```
docker run --name ubuntu-container -it ubuntu bash
```

You can execute additional commands in a running container with [docker exec](https://docs.docker.com/engine/reference/commandline/exec/). Open additional multiple terminals and run 

```
docker exec -it ubuntu-container bash
```

to start bash on the running container. You can now `touch` a file in one shell and verify the changed state with `ls` on another one. The file will still be there if you stop (`docker stop container_name`) and then start (`docker start container_name`) the container. But it will be gone forever if you remove the container.

## Step 2: Time to run a web server (10 min)

This repository creates everything you need to set up a simple web server: a `Dockerfile` and some content.

### Task

1. Have a look at the Python code in the `/app` directory. The `app.py` module contains a minimalist web application that uses the `flask` library.

2. Have a look at the `Dockerfile` and understand what it does.

2. Use the `Dockerfile` to build the image and give it the name `kwt22-image`
    ```
    docker build -t kwt22-image .
    ```

    Note that `.` refers to the current directory, i.e. call this from the root directory of the repository.

3. Congratulations, you built your first image. Now create and run the container with 
    ```
    docker run --rm -it --name kwt22-container -p 8080:5000 kwt22-image --host=0.0.0.0 --port=5000
    ```
    The arguments after `kwt22-image` are passed to the entrypoint of your container. 
    This will let the python web application listen on port 5000, which docker then maps to port 8080 ([http://127.0.0.1:8080](http://127.0.0.1:8080)) on your system.
    With `--rm`, we tell docker to remove the container when it exits.
    Wit `-it`, we [run](https://docs.docker.com/engine/reference/run/) the web application as an interactive process.

    Note that `--host=0.0.0.0 --port=5000` is already provided by default via

    ```dockerfile
    CMD ["--host", "0.0.0.0", "--port", "5000"]
    ```

    in the `Dockerfile`. You actually don't need it and can just call 

    ```
    docker run --rm -it --name kwt22-container -p 8080:5000 kwt22-image
    ```

4. You can investigate the status of the container with
    ```
    docker container ls
    ```
    in another terminal.

5. Optional: to stop the container, you can abort the current process in the terminal that is running the web server with `control+C` or execute `docker container stop kwt22-container`.

### [Optional Bonus Task]

Given that all steps are executed as described above, is there another way to access the website?

Hint: Investigate the debug output of Flask or execute

```
docker inspect kwt22-container
```


## Step 3: Let's develop with docker (10 min)

Next, we will use so-called `mount binds` to mount our code in a docker container. This is a simple way to develop with docker.
All our code is still on the local machine, but it is executed within docker.

The current `Dockerfile` copies everything in `/app` to the docker image. We don't want that anymore, so just comment out the line that says
```dockerfile
COPY ./app ./
```

To mount your local app directory, rebuild the image and run the container with 
```
docker run --rm -it --name kwt22-container -v "/$(pwd)/app":"/app" -p 8080:5000 kwt22-image
```
*Note that this creates the `app/__pycache__` directory and places the compiled Python byte code for your `app.py` module there. To this date, the management of file permissions with docker is still a bit tedious. If you are running Linux, the  files in `app/__pycache__` will be owned by `root:root` on the host system, as they are created by root in the container. Although the root user in the docker container is not the root user on the host system, you have to be root on the host to delete the files (or change permissions).*

### Task
Your task is to edit `app.py` on your host system however you like. First try small modifications like changing the returned string. You will find further inspiration in the [flask documentation](https://flask.palletsprojects.com/en/2.2.x/quickstart/). The flask development server will automatically reload the file when you modify it.


## Step 4: Getting ready for production (10 min)

The flask development server is pretty slow. While it makes development easy, we want to deploy our web app with a more sophisticated *Web Server Gateway Interface (WSGI)* server.


### Task

Create a simple multi-stage `Dockerfile` that allows to build development and production images. We will use [waitress](https://docs.pylonsproject.org/projects/waitress/en/latest/) in the hints and the solution, but you could also use [different WSGI servers](https://flask.palletsprojects.com/en/2.2.x/deploying/).

### Hints

To use the waitress server, you first have to [download](https://pypi.org/project/waitress/) it, e.g. with `pip`. You can then use the `waitress-serve` command like
```
waitress-serve --host=0.0.0.0 --port=5000 app:app
```

How does this map to your previous entrypoint and run command? How do you have to change them?

The following commands assume that your dockerfile is structured like this:
```dockerfile
# development image
FROM python:3.9-slim-bullseye as dev

# ... do some stuff

ENTRYPOINT ...
CMD ...

# production image
FROM dev as prod

# ... do some stuff

ENTRYPOINT ...
CMD ...
```

This allows you to build development and production images like
```sh
# development
docker build -t kwt22-image:dev --target dev .
docker run --rm -it --name kwt22-container-dev -v "/$(pwd)/app":"/app" -p 8080:5000 kwt22-image:dev

# production
docker build -t kwt22-image:prod --target prod .
docker run --rm -it --name kwt22-container-prod -p 8080:5000 kwt22-image:prod
```

## Congratulations

Congratulations, you are done! You now know..
* Dockerfile basics and how to build images
* How to run containers, container management basics
* A simple way to develop with docker
* How to create production images with multi-stage Dockerfiles
* Extra: how to set up a simple Python web app

Time to transfer this knowledge to your own projects!


## Workshop Extensions

### Version Upgrades

You wrote your code and executed it with the ancient Python version 3.9. Does it also work with Python 3.10? Have a look at [https://hub.docker.com/_/python](https://hub.docker.com/_/python) and modify the base image to run your web application with Python 3.10. You can also try a later release candidate if you feel adventurous. All by changing a single line.

### Docker in VS Code

While it is possible to do everything by hand, managing your development containers manually is a bit tedious. Many IDEs include extensions for remote development, including development with Docker containers.

The [.devcontainer/](.devcontainer/) directory contains everything you need to set up this project with VS Code.

