# Container_from_Code
Create a Python Container running python http.server and JS/HTML5 (HexGL) App.

## Description
I have been using the HexGL App to run demos for a few years now.  I have used this app through several iterations:

* First running/hosting on a "typical" Apache webserver
* following the author's example of using Python to test  
* (I then forked the project to adapt for my own needs)
* Updated to run in Red Hat OpenShift
* run as a simple local container (what this repo is about)

## Sources
[HexGL Homepage](https://hexgl.bkcore.com/)  
[HexGL GitHub](https://github.com/BKcore/HexGL)  
[My HexGL GitHub](https://github.com/KnowBetterCloud/HexGL)  

## Let's roll...
(preface) please keep in mind, I am certainly not a developer.  I am very much focused on infrastructure and core-services.  So, my perspective and approach at analysis and problem solving may look a bit unorthodox.

I reviewed BKcore's source on Github to determine what "type of app" I was working with.  I see some *.js files... some *.webapp - and then I look at how he suggests you run the app;

```
cd ~/
git clone git://github.com/BKcore/HexGL.git
cd HexGL
python -m SimpleHTTPServer
chromium index.html
```
ah - so I just need a webserver.  Cool.  (note:  I am running Fedora 37 which has "chromium-browser" - which you will see in my overview)

### Create a docker file
So - since I don't actually know what I am doing, I started by using the latest python (which is python3).  And the resulting change - instead of running "SimpleHTTPServer - we will run http.server.

Create a "Dockerfile"
```
from python:latest

WORKDIR /var/www/html
COPY * /var/www/html/

EXPOSE 8000
RUN echo "$PWD"
CMD python -m http.server 8000
```

### Build a Container

```
git clone git://github.com/knowbettercloud/HexGL.git
cd HexGL
podman build -t my-hexgl .
podman run --name hexgl localhost/my-hexgl
chromium-browser http://localhost:8000/index.html
```

 
