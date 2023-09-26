---
title: "Docker Tutorial"
date: 2023-03-14T08:10:40+02:00
draft: false
---

# Most basic Docker tutorial

Let's have a look at Docker! For this purpose we are going to run a simple Flask application from an Docker container.
At the end we will recap some basic terminology related to Docker. I have Docker already installed
and will not go through the installation process,
you can follow the official [Docker documentation](https://docs.docker.com/engine/install/). First we have to look
for a base image we can use for our purposes. We'll create a simple "Hello World" site with Flask,
a Python web framework, so let's search for an image on the [Docker Hub](https://hub.docker.com/).
We will use the official Python image:

```
docker pull python
```

Docker will now pull the latest Python image and after it's done we can check it with

```
docker images
```

If you don't have any other images installed the terminal output should look like this

```
REPOSITORY    TAG         IMAGE ID       CREATED       SIZE
python        latest      0a6cd0db41a4   13 days ago   920MB

```

We can now run our Docker image, but by default nothing will happen, we only have the base image. Let's give
it an extra command:

```
docker run python echo "Hello From Docker"
```
You should see "Hello From Docker" printed to your terminal. The Docker client ran our Docker container,
executed our command and exited after finishing, so we have no running Docker container.
We can check for running Docker containers with

```
docker ps
```

or

```
docker ps -a
```

if we want to see which containers we have run. In our case the output would be

```
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS                     PORTS     NAMES
36f40515bd7b   python        "echo 'Hello From Do…"   3 minutes ago   Exited (0) 3 minutes ago             nice_wozniak
```

We can keep our container running with, for example

```
docker run -it python
```

Now you should be inside the interactive mode of your Python installation inside your Docker container.
Feel free to try out some commands and exit with "exit()". In case you want to delete a specific container, just run

```
docker rm 36f40515bd7b
```

"rm" is the remove command followed by one or more container IDs. You can see the container ID in our previous example
when running "docker ps -a". There are more advanced ways to do this so you don't have to type every single
container ID by hand, but for the purpose of this tutorial this should be fine. If you want to delete all containers
you can run "docker container prune", you will be asked if you really want to delete all stopped containers so you
don't delete them all by mistake. If there are images you don't longer need you can delete them by running
"docker rmi". Now let's install Flask in our project folder, outside of our container. I created a folder
"docker_test", inside we run the following commands:

```
python3 -m venv venv
source ./venv/bin/activate
```

We created a virtual environment for our Flask app, now we install Flask itself and write our Hello World website:

```
pip install Flask
pip freeze > requirements.txt
touch hello.py

```
Inside the hello.py we write
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello from Flask!</p>"
```

We can now run "flask --app hello run" to run our Flask app and see an output like this

```
 * Serving Flask app 'hello'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
127.0.0.1 - - [05/Jun/2023 11:39:20] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [05/Jun/2023 11:39:20] "GET /favicon.ico HTTP/1.1" 404 -
```

We can copy the adress our app is running on, in our case "http://127.0.0.1:5000", to our browser and we will
see a simple "Hello from Flask!" on the page. Now it's time to set up our Docker file. First shutdown our Flask app
and create a file named "Dockerfile". Inside we write

```
FROM python:latest

# directory for the app
WORKDIR /usr/src/app

# copy files to container
COPY . .

# install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# define port
EXPOSE 5000

# run command inside the container
CMD ["flask", "--app", "hello", "run", "--host=0.0.0.0"]
```

and then run "docker build -t yourusername/catnip ." to build our image. Insert your own username, obviously.
After it's done we can run "docker run -p 8888:5000 yourusername/catnip" and see the following output

```
 * Serving Flask app 'hello'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000
Press CTRL+C to quit
```

If we go to our browser and visit 172.17.0.2:5000 we will see our page displayed! Now that we have our super fancy
website running, let's recap some terminology we used in our tutorial:

# Image
A base image we use to build our container. Examples include Ubuntu, Python, Busybox.
# Client
Client in this context means the Docker CLI tool which we use to interact with a daemon.
# Daemon
The service running on your own machine which manages Docker containers, including building, running and distributing
them.
# Container
A Docker container is similar to a virtual machine, just more lightweight. We use it to
provide a layer of abstraction/isolation from our OS and environment, so when we run our container our application
inside the container doesn't care about the OS it runs on (as long Docker itself is supported).
# Registry
A Docker registry hosts Docker images, in our case we used the official Docker Hub but you can use others, including
hosting your own registry.





