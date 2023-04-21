# Container_from_Code

I will provide a few examples of how to take existing code and make it run in a container.

Note: please keep in mind, my primary "persona" has not been "a developer".  I am very much focused on infrastructure and core-services.  So, my perspective and approach at analysis and problem solving may look a bit unorthodox.  

Create a Python Container running python http.server and JS/HTML5 (HexGL) App.

## Python container to run JS App

### Description
I have been using the HexGL App to run demos for a few years now.  I have used this app through several iterations:

* First running/hosting on a "typical" Apache webserver
* following the author's example of using Python to test  
* (I then forked the project to adapt for my own needs)
* Updated to run in Red Hat OpenShift
* run as a simple local container (what this repo is about)

### Sources
[HexGL Homepage](https://hexgl.bkcore.com/)  
[HexGL GitHub](https://github.com/BKcore/HexGL)  
[My HexGL GitHub](https://github.com/KnowBetterCloud/HexGL)  

### Let's roll...
I reviewed BKcore's source on Github to determine what "type of app" I was working with.  I see some *.js files... some *.webapp - and then I look at how he suggests you run the app;

```
cd ~/
git clone git://github.com/BKcore/HexGL.git
cd HexGL
python -m SimpleHTTPServer
chromium index.html
```
ah - so I just need a webserver.  Cool.  (note:  I am running Fedora 37 which has "chromium-browser" - which you will see in my overview)

### Create a docker file (Dockerfile)
So - since I don't actually know what I am doing, I started by using the latest python (which is python3).  And the resulting change - instead of running "SimpleHTTPServer - we will run http.server.

Create a "Dockerfile" [example Dockerfile](https://raw.githubusercontent.com/KnowBetterCloud/HexGL/main/Dockerfile)
```
FROM python:latest

LABEL description="HexGL Container Image"
LABEL version="1.0"
LABEL maintainer="knowbettercloud@gmail.com"

#CMD mkdir -p /var/www/html
WORKDIR /var/www/html
# COPY * /var/www/html/
COPY . /var/www/html/

# At Build Time
RUN echo "Note: App will be exposed on port: 8000"
# At Run Time
CMD echo "Note: App will be exposed on port: 8000"
EXPOSE 8000

#CMD python -m http.server 8000
CMD [ "python","-m","http.server","8000"]
```

### Build a Container

```
mkdir ~/Projects; cd $_
git clone https://github.com/knowbettercloud/HexGL.git
cd HexGL
podman build -t my-hexgl .
podman run -p 8000:8000 --name hexgl localhost/my-hexgl
```

Or.. Docker, if you prefer
```
mkdir ~/Projects; cd $_
git clone https://github.com/knowbettercloud/HexGL.git
cd HexGL
docker build -t my-hexgl .
docker run -p 8000:8000 --name hexgl my-hexgl
```

Then, in another terminal window, run:
```
chromium-browser http://localhost:8000/index.html
```

To "enter" the pod to review the content
```
podman exec -it hexgl /bin/bash
podman logs -f (pod number)
```

To "reset"
```
docker kill $(docker ps -a | grep hexgl | awk '{ print $1 }')
docker rm $(docker ps -a | grep hexgl | awk '{ print $1 }')

podman images prune
```

as an example, check this out
```
podman run -p 8081:80 acantril/containerofcats
```

You may need to run chromium to get it working (I think because of permissions, supportability, etc..)
```
chromium-browser http://localhost:8081/index.html
```

## Optimization (peeping around corners)
To quote one of my favorite old school rap artists... "I keep looking over my shoulder and peeping around corners"  

storyline:  While troubleshooting my erroneous "COPY syntax" - I discovered that the "python:latest" image was 956MB.  Whoa!  So - I did some research and discoverd a mention of ["3.8.2-slim-buster"](https://hub.docker.com/layers/library/python/3.8.2-slim-buster/images/sha256-a6e1e46966d0c386381ec4f0c6021db36b24340df133e9f54f2a21c0941ffbae) - hmm.. let's give it a shot.

Updated my Dockerfile from 
```
FROM python:latest
```
to
```
FROM python:3.9-slim-buster
```
and it seems to (still) work.  #solid

