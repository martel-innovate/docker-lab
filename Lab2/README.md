# Lab - 2

## Running your first docker application

### Create your first image descriptor
Create an empty directory. Change directories (cd)
into the new directory, create a file called ``Dockerfile``,
copy-and-paste the following content into that file, and save it.
You can browse for existing official images on Docker Hub. Usually if the official image is based on Alpine Linux, its footprint is much smaller.

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]```

This Dockerfile refers to a couple of files we haven’t created yet,
namely ``app.py`` and ``requirements.txt``.

## The app itself
Create two more files, ``requirements.txt`` and ``app.py``,
and put them in the same folder with the ``Dockerfile``.
This completes our app, which as you can see is quite simple.
When the above Dockerfile is built into an image,
``app.py`` and ``requirements.txt`` will be present because of that
``Dockerfile``’s ``ADD`` command, and the output from ``app.py``
will be accessible over HTTP thanks to the ``EXPOSE`` command.

* requirements.txt:

```
Flask
Redis
```

* app.py:

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

Now we see that ```pip install -r requirements.txt``` installs the Flask and Redis libraries for Python,
and the app prints the environment variable ```NAME```, as well as the output of a call to ```socket.gethostname()```.
Finally, because Redis isn’t running (as we’ve only installed the Python library, and not Redis itself), we should expect that the attempt to use it here will fail and produce the error message.

## Build the app image
Run the build command. This creates a Docker image:

```
docker build -t friendlyhello .
```

Let's check that the image was built correctly:
```
$ docker images

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```

## Run the app
Run the app, mapping your machine’s port 4000 to the container’s published port 80 using -p:
```
docker run -p 4000:80 friendlyhello```

You should see a notice that Python is serving your app at ```http://0.0.0.0:80```.
The message is coming from inside the container, which doesn’t know you mapped port ``80`` of that container to ```4000```, making the correct URL ``http://localhost:4000``.

Go to that URL in a web browser to see the display content served up on a web page, including **Hello World** text, the **container ID**, and the **Redis error message**.

Hit ``CTRL+C`` in your terminal to quit.

Now let’s run the app in the background, in detached mode:

``docker run -d -p 4000:80 friendlyhello``

You get the long container ID for your app and then are kicked back to your terminal. Your container is running in the background. You can also see the abbreviated container ID with ``docker container ls`` (and both work interchangeably when running commands):

``$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago```

Now use docker stop to end the process, using the CONTAINER ID, like so:

``docker stop 1fa4ab2cf395``

## Share your image
To share the image we just created, let’s upload our built image and run it somewhere else.

A registry is a collection of repositories, and a repository is a collection of images—sort of like a GitHub repository, except the code is already built. An account on a registry can create many repositories. The docker CLI uses Docker’s public registry by default.

### Log in with your Docker ID
1. If you don’t have a Docker account, sign up for one at cloud.docker.com. Make note of your username.
2. Log in to the Docker public registry on your local machine:
```docker login```

### Tag the image
The notation for associating a local image with a repository on a registry is ```username/repository:tag```. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as ```get-started:part2```. This will put the image in the get-started repository and tag it as part2.

Now, put it all together to tag the image. Run docker tag image with your username, repository, and tag names so that the image will upload to your desired destination. The syntax of the command is:
``docker tag image username/repository:tag```

For example:
```
docker tag friendlyhello john/get-started:part2```

Run ``docker images`` to see your newly tagged image. (You can also use ``docker image ls``.)

```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```
### Publish the image
Upload your tagged image to the repository:

``docker push username/repository:tag``

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you will see the new image there, with its pull command.

### Pull and run the image from the remote repository
From now on, you can use docker run and run your app on any machine with this command:

``docker run -p 4000:80 username/repository:tag``

If the image isn’t available locally on the machine, Docker will pull it from the repository.

```
$ docker run -p 4000:80 john/get-started:part2
Unable to find image 'john/get-started:part2' locally
part2: Pulling from john/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/get-started:part2
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

## Monitor you environment

CAdvisor is a small addon that can help you to monitor your docker services.

To launch it:
```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
``
