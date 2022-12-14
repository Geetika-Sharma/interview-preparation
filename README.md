# interview-preparation

This repository contains notes related to DevOps interview

## Docker Questions
### How will you create a Dockerfile that prints the runtime string passed to it? 
e.g
> docker run print_args Hello World

> Hello World

Answer:
Create a Shell Script and Dockerfile as follows:
vim print.sh
```
#!/bin/sh
 echo "$@"
```

vim Dockerfile
```
FROM alpine
COPY ./print.sh /
RUN chmod +x /print.sh
ENTRYPOINT ["/print.sh"]
```
Now build and run the container 
```
docker build -t print_args .
docker run print_args Hello World
```
