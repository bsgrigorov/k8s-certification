# Dockerfile
You may be required to build a dockerfile in the exam. You will have access to dockerfile docs. 

Instruction - Argument
```
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql 

COPY . /opt/source-code

ENTRYPOINT FLASK APP=/opt/source-code/app.py flask run
```

```
docker build -t tagname .
```

Layered architecture 
- layers are cached - if there is a failure in a layer it will restart on that layer and reuse the successful ones
- rebuilding images is faster that way
- only layers above a change need to be rebuilt

```
docker history taggedimage
```

Get base image OS
```
docker run <image-nme> cat /etc/*release*
```

Run container
```
docker run <image-name> --name <container-name> -d 
```