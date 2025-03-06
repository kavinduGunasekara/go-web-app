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

**Multistage Docker Build Successfull**

**Step Two**
write k8s.yaml manifest so project is also ready to  deployed on k8s cluster

push image to docker repository (Hub)
why do this 
  *whenwrite k8s deployment out of the box behavior of that k8s cluster will try to pull the image from image registry
of cours you can use loacl image but better practice is pull the image from docker image registry then for this first we need push the image to registry

  **push image to docker repository (Hub)** 
 
 ```bash
$ docker push kavindu200/go-web-app:v1
$ docker run -p 8080:8080 -it kavindu2000/go-web-app:v1
port mapping (what is 8080:8080 that map conatiner port with host machine port )

```
Write K8s file

Now create go to go-web-app folder create folder called **k8s**
inside another folder manifest
inside it write deplyment.yaml(also you can deploy application as a pod but if we deployed it as pod then you do not get the capabilites a replicaset controller a deploment so always go with writhing a deployment file)

2.1 deployment.yaml file write

to write use K8s doc --> https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

example deplyment.yaml
![3write k8s deploymentYAML](https://github.com/user-attachments/assets/c245c7b8-55d3-4cad-a361-b225c8787ccb)

 ```bash
# This is a sample deployment manifest file for a simple web application.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-web-app
  template:
    metadata:
      labels:
        app: go-web-app
    spec:
      containers:
      - name: go-web-app
        image: kavindugunasekara2000/go-web-app
        ports:
        - containerPort: 8080

```

 done deployment.yaml

 2.2 service.yaml file write
 inside manifest create service.yaml file
 
go to k8s service doc - https://kubernetes.io/docs/concepts/services-networking/service/

service.yaml
![4write k8s servvice YAML](https://github.com/user-attachments/assets/c7ca469d-8580-46c7-bfad-41bc258de28a)


 ```bashf
# Service for the application
apiVersion: v1
kind: Service
metadata:
  name: go-web-app
  labels:
    app: go-web-app
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: go-web-app
  type: ClusterIP
```
2.3 ingress.yaml
inside manifest create ingress file

go to k8s ingress doc - https://kubernetes.io/docs/concepts/services-networking/ingress/

** we use host based ingress** whre we will route application  traffic we will use perticular host name and we will only allowed the application traffic on that particular hostname as example you can access amozon.com application only when you hit the amozon .com  so there is DNS mapping which we will understoof towards the end of this project


ingress.yaml 
![5go to k8s ingress yaml](https://github.com/user-attachments/assets/a3967d87-5863-4a82-910d-93bcbad33a53)

 
 ```bashf
# Ingress resource for the application
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: go-web-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: go-web-app.local
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: go-web-app
            port:
              number: 80
```

 ```bashf
what is ingress file -->In Kubernetes, an Ingress file is used to manage external access to services within a cluster, acting as a single entry point for incoming traffic and routing it to the appropriate services based on defined rules, essentially providing a way to securely and efficiently expose multiple services to the outside world through a single IP address while controlling how traffic is distributed among them; this is particularly valuable when you need complex traffic routing or want to leverage features like load balancing and SSL termination
```

**in above code what is ingress class name**
it is basically for the ingress resources to be identified by ingress controller in some project in some organization there can be mutiple ingress controller so some company use nginx, AWS application load baalancers  becasues muliple development team ask for multiple ingress controllers now ingress resources with in the k8s cluster need to be identified by those ingress  controllers , then what we usually do we provide a ingress class name and this ingres class name tells the nginx ingress controller that this is the ingress resources that you have to watch first , if you not providing this first the nginx ingress controler will not watch your ingress resources.


In this we use nginx ingress controoler


2.4 now validate k8s manifest that we have written 


2.4.1 we need have a k8s cluster
2.4.2 we use EKS from AWS (it is enterpirse k8s cluster)f
2.4.3 How to run Eks cluster 

check thos things 
step o1

```bash
      prerequisites
kubectl – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

eksctl – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

AWS CLI – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.

use for authentication with AWWS account
f
```

logiing AWS with IAM user and provide eks cluster policies like create , delete adn u need to grant the access

but you can use simply root user 
authenticate AWS with your loacl machine 

 go to loacl machine
   aws account go to securty credentiaal and go to access key and create
   run $ aws configure 
   then ask access key
   security access key 
   default region
   output format as json

now authenticate
--------------------------------------f
**Install EKs cluster

   step 02

```bash
Install EKS
Please follow the prerequisites doc before this.

Install a EKS cluster with EKSCTL
   $ eksctl create cluster --name demo-cluster --region us-east-1 
Delete the cluster
   $ ksctl delete cluster --name demo-cluster --region us-east-1
```
![6create eks cluster](https://github.com/user-attachments/assets/8f73b875-33db-437e-8d19-1bffbac5c022)


take k8s manifest

2.5 Start with deployment.yaml first
```bash
$ j=kubectl apply -f k8s/manifest/deployment.yaml
```
![7create depolyment maifest](https://github.com/user-attachments/assets/eefb1102-1eac-4a90-936a-b9840db1e322)

now go-web-app is running , pod is running succesfully

![8pod running succeesfully](https://github.com/user-attachments/assets/fe27ce9a-6ebf-4c73-a54f-1f25dca4fb61)

2.6 verify if my service is implemented corectly

```bash
$ j=kubectl apply -f k8s/manifest/service.yaml
```

2.7 proces with ingress

```bash
$ j=kubectl apply -f k8s/manifest/ingress.yaml
```

![9create ingress and service YAML](https://github.com/user-attachments/assets/68742458-b8a9-4ec7-9718-429c4b7d26a4)

now i can not access my resources directly from the ingress becauses 
type 
```bash
$kubectl get ing

```
![10create ingress file](https://github.com/user-attachments/assets/319cdf8f-9985-487a-a3bd-f656f1d5c243)

then you willl see adresssss not assigned
what we need for that we need ingress controller assign the address for the ingress resource
onecs the address is assigned we can take the IP address and we can map it with tha domain name
that we have created in my etc host
before it verify at least service is working fine
then what we can do we cam expose the service in the nodeport mode
then before move forward verify service is working fine

   


