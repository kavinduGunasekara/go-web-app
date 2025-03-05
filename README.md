# Go Web Application

Hello 

Devops Implementation

1. Containerization with multistage docker file (implement images with reduce images as well as security)
2. K8s manifest such as deplyment ,serivce , ingress
3. Setup Ci using Github Action
4. Cd using GitOps ( using ArgoCd to implement continous delivery)
5. k8 cluster set up beacuse ci cd has to deploy applicaion on the target platform then it is K8s(then application deploy on EKS cluster)
6. set up helm chart Because that use full when dev team need to deploy application on Develop, QA or Production and muliple environment instend of writing manifest for each environment then they can use helm chart pass the values.yaml

7. set ingress controller configuration so that would create loadbalancer depending upon ingress configuration -> so that application expose to the outside world

8. how to map load balancer ip address to loacl DNS so that we test if the application is access from the outside world


**Strt the Project**

**Step one**
Containeriztion the project

first understand 
1.application how the applcation should be build
2.how the application sholud be run on which port application running
3.how the application looks like

befre containerize runn application locally
clone  the repo and run loaclly

```bash
git clone https://github.com/iam-veeramalla/go-web-app/tree/main
cd go-web-app
go build -o main . ## sholud build before run

go run main.go
```
The server will start on port 8080. You can access it by navigating to `http://localhost:8080/courses` in your web browser.

## Looks like this

**this is application**
![1run continer loaclly](https://github.com/user-attachments/assets/1c8f72f3-7b1f-401e-ac81-421524d3f269)



**Bold Text**
__Start write Docker file__

step 01
create Dockerfile(write multistage docker file)
advantage of multistage 
   1.Stage one-  build docker image( download all dependencies use any base image)
    after build 

   2.Stage two
   you can use distroles image as base image ( it has security and reduce image size)
   copy binary build in satge one 
   expose port 
   run application

**Dockerfile**
   
```bash
# Containerize the go application that we have created
# This is the Dockerfile that we will use to build the image
# and run the container

# Start with a base image
FROM golang:1.21 as base

# Set the working directory inside the container
WORKDIR /app

# Copy the go.mod and go.sum files to the working directory
COPY go.mod ./

# Download all the dependencies
RUN go mod download

# Copy the source code to the working directory
COPY . .

# Build the application
RUN go build -o main .

#######################################################
# Reduce the image size using multi-stage builds
# We will use a distroless image to run the application
FROM gcr.io/distroless/base

# Copy the binary from the previous stage
COPY --from=base /app/main .

# Copy the static files from the previous stage
COPY --from=base /app/static ./static

# Expose the port on which the application will run
EXPOSE 8080

# Command to run the application
CMD ["./main"]
```

 go to terminal to test Dockerfile
 (need install docker your local )
 ```bash
$ docker build -t -dockerusername-/go-web-app:v1 .
$ docker run -p 8080:8080 -it kavindu2000/go-web-app:v1
port mapping (what is 8080:8080 that map conatiner port with host machine port )

```
**Now it running well continerize successfull**
![1run continer loaclly](https://github.com/user-attachments/assets/73c53ab3-632f-473d-8232-e1a8a00b4770)

 










   


