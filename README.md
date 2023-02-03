# Container_from_Code

I will provide a few examples of how to take existing code and make it run in a container.

Note: please keep in mind, I am certainly not a developer.  I am very much focused on infrastructure and core-services.  So, my perspective and approach at analysis and problem solving may look a bit unorthodox.  


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

Create a "Dockerfile"
```
FROM python:latest
LABEL description="HexGL Container Image"
LABEL version="1.0"

CMD mkdir -p /var/www/html
WORKDIR /var/www/html
COPY * /var/www/html/

EXPOSE 8000
CMD "Note: App will be exposed on port: 8000"
CMD python -m http.server
```

### Build a Container

```
git clone https://github.com/knowbettercloud/HexGL.git
cd HexGL
podman build -t my-hexgl .
podman run -p 8000:8000 --name hexgl localhost/my-hexgl
chromium-browser http://localhost:8000/index.html
```

```
docker kill $(docker ps -a | grep hexgl | awk '{ print $1 }')
docker rm $(docker ps -a | grep hexgl | awk '{ print $1 }')
```


as an example, check this out
```
podman run -p 8081:80 acantril/containerofcats
```
chromium-browser http://localhost:8081/index.html
 
