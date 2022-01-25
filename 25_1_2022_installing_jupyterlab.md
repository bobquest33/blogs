# Setting JupyterLab With Docker & Raspberry Pi 4

*Date: 25-Jan-2022*

## Background

I have been trying to find a suitable Web IDE for my projects which I can use directly from browser. I found some websites and some truly cloud IDEs but nothing that can run out of docker on my PC or on Raspberry pi.

Some examples of popular pure cloud IDEs are: 
- ShiftEdit
- Cloud9
- Eclipse Che
- replit
- Github codespaces

Also I found IceCoder, which is more of a Web editor, but it was not a real IDE. I wanted an IDE which was not only a web editor but also share a terminal and support executing multiple backend. In this only jupyterlab fit the bit. While I have use jupyter notebook and jupyter lab in docker earlier but ws not sure if it will run on my raspberry pi 4. While I expected it to be few hour work, getting the right Dockerfile setup itself took me a whole day and was a frustrating and learning experience. I am writing this blog to capture my experience and also share some of the challenges I faced in setting up the environment.


## Envionment Setup

### Installing Docker

For this POC I already had installed Docker on my Raspberry Pi 4, you can get the instructions in below link.

https://docs.docker.com/engine/install/debian/

I use docker compose to manage the environment dependencies, I had installed docker-compose directly from pip using the below command https://pypi.org/project/docker-compose/

```
pip install docker-compose
```

*Note: The New compose is being integrated into the docker command and that may change things in future*

### The Folder Structure

I used the following folder structure as I felt I may be installing other dependencies my docker-compose file in future. Feel free to change to your needs

```

- docker-compose.yml
- docker
    - jupyter
        - Dockerfile
        - entrypoint.sh
- notebooks
```

`docker-compose.yml` -> the compose file to note the services, volumes and dependencies to setup my docker environment
`Dockerfile` -> This contains the reference to base image the steps for installing Jupyterlab and running it.
`entrypoint.sh` -> This is the entry point script that contains the logic to start Jupyterlab process

### Dockerfile

The `Dockerfile` looks as below

```
FROM balenalib/raspberry-pi-debian-python:3-buster

RUN apt-get update && apt-get -y update
RUN apt-get install -y build-essential python3-dev libffi-dev

RUN pip3 install --no-cache-dir jupyterlab

RUN mkdir /work

COPY ./docker/jupyter/entrypoint.sh /work/entrypoint.sh

RUN mkdir /work/notebook

WORKDIR /work/notebook

EXPOSE 8081
RUN chmod -R 765 /work/*.sh
ENTRYPOINT ["/work/entrypoint.sh"]
CMD ["server"]
```

The most important part was selecting the base image `balenalib/raspberry-pi-debian-python:3-buster` I tried multiple base images from Ubuntu, to `python3-slim` and even the latest Debian version, but there seems to be a continuing problem is Debian Bullseye version which seems to break pip.

The second most important part was to install `build-essentials` and `python3-dev` as there are many platform specific components in jupyterlab for which `gcc` and `g++` will be required during the build process.

I had setup the install folder as `/work` and created a new folder called `/work/notebook` where all notebooks can be kept and exposed as a volume

### entrypoint.sh

The `entrypoint.sh` looks as below, this is a format I have used from my earlier formats, its simple and works for me

```
#!/bin/bash
set -e

echo "Container's IP address: `awk 'END{print $1}' /etc/hosts`"

if [ "$1" = 'server' ]; then
   jupyter lab --port=8081 --no-browser --ip=0.0.0.0 --allow-root
else
    exec "$@"
fi
```

The main part in this script is to point `--ip` to `0.0.0.0`, this is to make sure the Http connections from outside the container can hit the Jupyterlab  process. While this may be obvious to other developers, it was a pain to me when it was pointing to `localhost` by default and connections were not hitting Jupyterlab 

### docker-compose yml file

The `docker-compose.yml` is also very simple where only one service `jupyterlab` is defined.

```
version: '3'
services:
  jupyterlab:
    build:
      context: .
      dockerfile: ./docker/jupyter/Dockerfile
    image: "bobquest33/jupyterlab:latest"
    restart: always
    user: root
    volumes:
      - /path/to/jupyterlab/notebooks:/work/notebook
    ports:
      - 8081:8081
    container_name: jupyterlab-container
```

Under `build` tag, the `context` folder path and the `dockerfile` path are important as they point to the current path from where you will be executing the docker-compose.

The section under volumes is important as there I map my local notebooks folder to the path `/work/notebook` under the container so that any files or notebooks created or modified under jupyter container are persisted on the host system disk.

### Running Jupyterlab 

Once we are ready, we can run the Jupyterlab  with the below command from the root directory where `docker-compose.yml` is present

```
docker-compose up --build -d
```

The `-d` option makes the Jupyterlab  service run in the background and the `--build` flag ensures that if any changes are made to the `Dockerfile` the Jupyterlab  container image is built.

As a next step you will have to run `docker-compose logs` to check the URL for Jupyterlab , it will look as below

```
 To access the server, open this file in a browser:                                      
     file:///root/.local/share/jupyter/runtime/jpserver-9-open.html                      
 Or copy and paste one of these URLs:                                                    
     http://1a84add9fdd7:8081/lab?token=3078938b3dbe06114e11bfd53b91568bbaaf20cb7bc382cd 
  or http://127.0.0.1:8081/lab?token=3078938b3dbe06114e11bfd53b91568bbaaf20cb7bc382cd    
```

URL format to put in web browser will be `http://<raspberry ip address>:8081/lab?token=<url token value>` 

e.g. `http://192.168.11.21:8081/lab?token=3078938b3dbe06114e11bfd53b91568bbaaf20cb7bc382cd`

## Challenges & Solution

### Base Docker Image

When I started this POC, I thought this will be a quick 1 hour thing, but soon it turned to a whole night and day's worth of work. First big challenge was selecting the base image for Jupyterlab, first I tried many base docker images from [Belina](https://hub.docker.com/u/balenalib), but one of the most toughest part was getting any additional library installed, installing `wget` was getting challenging. 

First I tried installing using Jupyterlab  using `conda`, but installing `Miniconda`, proved challenging on many of the base images. Finally I settled to installing `balenalib/raspberry-pi-debian-python` base image, but again its latest version which points to Debian "Bullseye" where pip broke with `time.time()` with error `PermissionError: [Errno 1] Operation not permitted`,  detailed description below:

```
from pip._internal.utils import _log
File "/usr/local/lib/python3.10/site-packages/pip/_internal/utils/_log.py", line 8, in 
<module>
import logging
File "/usr/local/lib/python3.10/logging/__init__.py", line 57, in <module>
_startTime = time.time()
PermissionError: [Errno 1] Operation not permitted
```

More details in the given [link](https://stackoverflow.com/questions/70195968/dockerfile-raspberry-pi-python-pip-install-permissionerror-errno-1-operation
).

As a solution I pointed to the Debian buster version ` balenalib/raspberry-pi-debian-python:3-buster` of the base image, things proceeded fine. After the Docker file was updated and simplified, and the Jupyterlab was compiled.

### Installing pip dependencies

The next step that took longest time for me was figuring out what Debian packages to install, earlier I did the mistake upgrading pip in the Dockerfile, this prevented even to install Jupyterlab, then I have to install `buildessentials` and `python3-dev` to make sure the native dependencies of the dependent packages of pip gets installed.

### Bash script format

Lastly, the container did not start, I finally realised that the `entrypoint.sh` that I have developed was have `CRLF` which is DOS format, it should be in Unix format converting the file to Unix using `dos2unix` helped.

```
dos2unix entrypoint.sh
```

## Next Steps

Overall this was a great experience, I learnt many things about how package a web product on Raspberry pi and how its not as straight forward as installing on a `x86` Windows or even a Mac machine and how I need to spend more time improving my docker expertise.

## References

- https://devrix.com/tutorial/web-ides-future-coding/
- https://icecoder.net/
- https://u.group/thinking/how-to-put-jupyter-notebooks-in-a-dockerfile/
- https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html
