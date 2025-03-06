# Go Web Application 🚀

Hello  
![Screenshot 2025-02-27 122649](https://github.com/user-attachments/assets/600eb845-be78-485b-b0e3-58b9e515c4ee)

**Complete DevOps Implementation in one project**

We will implement End to End DevOps on a Golang web application. We will implement the following things -->


![1 1cover photo](https://github.com/user-attachments/assets/ab9ba952-0405-4b52-9e5e-916468e520f0)

 
1. 🐳 Containerization with multistage docker file (implement images with reduce images as well as security)
2. ☸️K8s manifest such as deplyment ,serivce , ingress
3. ⚙️ Setup Ci using Github Action
4. 🚀 Cd using GitOps ( using ArgoCd to implement continous delivery)
5. ☁️ k8 cluster set up beacuse ci cd has to deploy applicaion on the target platform then it is K8s(then application deploy on EKS cluster)
6. 📦  set up helm chart Because that use full when dev team need to deploy application on Develop, QA or Production and muliple environment instend of writing manifest for each environment then they can use helm chart pass the values.yaml

7. 🌍 set ingress controller configuration so that would create loadbalancer depending upon ingress configuration -> so that application expose to the outside world

8. 🔗 how to map load balancer ip address to loacl DNS so that we test if the application is access from the outside world


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
then before move forward with the ingress controller configuration, verify service is working fine.
go to the EC2 first - beacuse if we are running the service in the node portmode  
so first edit the service 
```bash
kubectl edit svc go-web-app

```
then in end of opend file you will see there is type as clusterip then change it to nodeport

now going to verify service is woring fine or not , even without ingress configuration
then expose the service on node port mode then run this command to check

```bash
kubectl get svc
```
then it show go-web-app running on node port and can access on port 32271 on my node ip  address

![11node port change successfully](https://github.com/user-attachments/assets/f2c7487e-ec5d-4756-9068-60a6df2805bd)

 check your all  node ip address 

 ```bash
kubectl get nodes -o wide

```

then it show two nodes and those are external Ip address so then pick up one of them and using 
browser usign that ip and using node port then access to website

54.160.199.111:32271/courses

**Application running fine even on a k8s cluster**

![12connect to eks cluster](https://github.com/user-attachments/assets/62010144-e109-409a-ba01-912cf463479b)


then in this we have used service of type nodeport mode then what we have to do is

we done doeckr file
k8s(service ,deployment , ingress) manifest creation.
completed EKS cluster creation.
implement ingress controller as  well
now we will compleate ingress controller configuration

so the ingress controller configurattion

how to create ingresss controller creation  then go to doc -> search nginx ingress controller 
there are two 
1. proived by nginx
2. community driven nginx

go to this community driven nginx
go to aws section

then copy networkload balancer

 
Install Nginx Ingress Controller on AWS
Step 1: Deploy the below manifest

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml

```
run aboove command it will create ingress controller resources

three things remember about ingress
1.ingress itself
2.ingress controll 
3.load balancer

ingress controller will watch ingress resourc and create load balancer

in k8s cluster you can not create load balancer by your self because it is not practically 
easily to configure load balancer every time
then you can write ingress file
ingress controller which is implement usually a go program written by the load balancer company 
in this we use nginx , which is written by community.
ingress controller tht is basicaly a go program that is written by load balancer caompany what 
it does it will watch for the ingress resource  and create load balancer as per ingress 
configuration.


now see nginx ingress controller running 

![13 create iginx ingress contorller](https://github.com/user-attachments/assets/e61a4979-87cd-4434-838e-8bd42e0b0fd6)


```bash

kubectl get pod -n ingress-nginx

```

then you will find k8s pod  


 kubectl edit pod <-name of runnign nginx continer->  -n ingress-nginx

 in this oped file go down and check section ingress calss = nginx
 because in our ingress file we mention it , then this is how ingress cls name is used
 
 like this --

   ![14check ingreess controller](https://github.com/user-attachments/assets/78322f9a-251f-4a80-984e-9affbc30b320)

we have the ingress controller running lets see if this ingress controller was abale to to watch 
our ingress resource run that code to check

```bash
kubectl get ing
```

so the ingress resource is watch by the ingresss controller and it gave us a ip addresss actually
a domain name

like this --->

![15ensure load balancer](https://github.com/user-attachments/assets/8ab2ce9a-a3da-4d86-a68c-9a0f9a6ab682)

explain agaim
ingress controller watch for the ingress resouce and create a load balancer
so this nginx ingress controller watched for the ingress resource and created a load balancer
on AWS which is go to AWS check if it has create load balancer

check on AWS

![16 load balancer created by ingress controller](https://github.com/user-attachments/assets/00c4f855-d625-4c69-a898-85c8ac827fe2)

it creates load balancer succesfully that create by the ingress controller 
lets try usinf above photo show the DNS name using that check if i can accss

first time you have not access show 404 error

![16 1](https://github.com/user-attachments/assets/f3b0c64f-ed1d-484b-8dd3-c8d2e2d0198f)

then why can not access
 go to ingress file you have clearly explain the load balancer only request if someonr is 
 accessing the host name go web app.local

 in this check
 ![17 1](https://github.com/user-attachments/assets/ff28c69a-2e6f-41b6-bc7f-04c2149e63f0)

what we do in organiztion
, we provide something like amozon.com whare DNS is already mapped but in our case it not 
possible.

then what we will do we will get this host name


then first get loadbalancer address 
that one see in picture

![15ensure load balancer](https://github.com/user-attachments/assets/2e02e608-dddc-4560-91ee-def8b06d9cf0)

copy address and go to cmd
 type
```bash
nslookup above coppied address paste here

```
then you see the ip address then copy it 

In mac or linux machine

now go to 
```bash
sudo vim etc/hosts
```
In windows 

no go to hosts file using

```bash

cd C:\Windows\System32\drivers\etc
vim hosts
```

then now will open hosts file 
on it we are doing DNS mapping

follow like this

![18 1 1](https://github.com/user-attachments/assets/851b32f5-fee5-4b44-aac5-ea902cc3b3ea)


https://github.com/user-attachments/assets/e1d21ab8-efbd-49a1-8fef-8e3c05861d8d


Now go to browser and type **go-web-app.local/home**

**Now can acceess the application**
 ![18success access by gowebapp on url](https://github.com/user-attachments/assets/6fd9b95b-96a4-4673-acbd-29d01468105d)



**remember**
 in this we use go-web-app.local but in organiztion we will use like daraz.lk , google.com like that


 **Step 03**

 **Helm creation before Ci implement**


Helm is that when you want to deploy your application to different environments then , now
I have deplyment, service ingress files .. then alll the things on it hard coded

then when comes to developer environment as Dev for the staging environment I have QA 
for the production environemnt I have Production Team
then I will create folder like k8s dev, k8s Qa, for all above I  metioned ,
insted you can use helm also use customize and what you can do just , you can asked your develpoers right in the helm chart you can variablize these kind of things .
you can just say your development team or testing team whoever using it you can just tell them that you know pass the tag name as variable.

1.Create folder called HELM

![20create help folder](https://github.com/user-attachments/assets/bf444828-57fc-4b2a-adcc-38e31a81b27f)

2. Insall  helm locally
![21chek helm version](https://github.com/user-attachments/assets/659131f4-0952-4f34-a197-f820cd5ebfb1)
 

3.create helm chart in helm directory 

```bash
cd helm
helm create go-web-app-chart 
```

![23cratea helm chart in right directory](https://github.com/user-attachments/assets/fece1ccd-fe1e-4e40-a786-34d7ff0b797d)

4. go to go-web-app-chart

5. remove chart folder in that directory

![23 1](https://github.com/user-attachments/assets/63158cd5-c9bf-464f-8547-e30b895718dc)

now only there this files
chart.yaml templates values.yaml

6. remove every thing in the templates folder
```bash
rm -rf '
```

7. inside templates folder copy k8s folder and paste in templates folder
8. go to vim deployment.yaml folder and in that replace tag like this

![26 1](https://github.com/user-attachments/assets/b00cecd4-a634-4f33-b294-e5c6bab469da)


this mean helm  whenever executed will looks tag from the values.yaml file how  

go to code -> go helm-> go values.yaml and replace it with this code

```bash
# Default values for go-web-app-chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: kavindugunasekara2000/go-web-app
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "13573617143"

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
```

from this deployment yaml file get images and go to tag 

then when we implemtn cicd will update the helm values .yaml  dynamically  so every time cicd is 
run  what we are going to do we are update the values.yaml of helm with the latest image that we
created in the ci and using ARGOCD that latest image with the latest tag will be automaticaaly deployed. 


9. Verify helm worikng fine

first delete deployment , service , ingress file

![28 1](https://github.com/user-attachments/assets/4533a628-b8d3-4b3b-af66-0a1c596d253e)

now check using
```bash
kubectl get all
```
nothing inside you can see means nothing related to our go web app

10. now using HELM installl what we delete before

```bash
helm install go-web-app ./go-web-app-char
```
 ![26install every thing using helm](https://github.com/user-attachments/assets/039e9021-a20e-4394-b4d9-a01b8c39f325)


**what we done we have install samething using HELM**


11. run to verify
```bash
kubectl edit deploy go-web-app

```
then you will see the images section image namehas :v1

now check inside the templates  -> go to deployemnt.yaml has this one 
image: kavindugunasekara2000/go-web-app:{{ .Values.image.tag }}

when you run helm install command what HELM is done  it went to the deployemnt it saw that there is a substitution so it went to the values.yaml and it replaec the tag on the deployment.yaml
thats why in kubectl deployment we have version tag as v1


12. **HELM pert is done**
then uninstall helm every thing
```bash
helm uninstall go-web-app

```
![27remove helm after done the helm part alsp](https://github.com/user-attachments/assets/c719b716-9449-4b67-96b7-dcf73f7df3be)


13. **HELM pert is done**


**Step 04 Implemet Ci using Github Action**

**CI**

In ci we implement multiple stages
Steps
1.Build & Test(unit)
2.Static code analysis
3.create & push Docker image
4.Update HELM with docker image that we have created

  after this CD comes to the picture use ARGOCD
What we will do in CD Ones HELM tag is updated ARGOCD will pull the HELM chart and deployed it on to the K8s cluster ,

we implement CI and CD we use GIthub Action for Ci
and for cd we use Argocd
ci cd - where whenever developer commit a change or create a pull request (in this we developer 
commit ) whenever developer commit a change the cicd pipeline will be trigggered where as part 
of the ci  using github action we will run mutiple stages 
 **stages**
  1.Build and unit test
  2.static code analysis (to verify if code has any static code )
  3.Docker image create & push
  4.Update Helm also update K8s manifest 
  what we actually update is when the the docker images is connected here for this perticular 
  commit that developer has made know the docker image that we create will be mapped with the 
  developer commit , so the docker image will have a new tag and that will be updated in the 
  helm values.yaml 

* Every time developer makes a commit HELM is automaticaly updted the values.yaml with a new
  commit.
  and
  **Now Cd comes** to connect with it , where we will use ArgoCd will watch the helm chart
  whenever the values.yaml is updated it will pull the helm chart and install it on the k8s
  cluster , if the helm chart is already there it will update the helm chart so anew version is
  deployed.
  If you understand this workflow implementing ci cd is not difficult.

** what is important is you need to how CI and CD are connected you can add n number of stages 
inside your Ci that really does not mattter  because adding more stages is not difficult .

what difficult is how ci every new commit is  updated to your HELM and how CD immediately picks
up that change and deploys that on to the k8s cluster .

** NOW START IMPLEMENT ** 
Let's have a FUN


**STEPs --->**
step 01
1.create a folder as GitHub inside this create another folder workflows
2. inside workflow  create ci.yaml 
-->
![29 1](https://github.com/user-attachments/assets/79aeac7f-b1c6-46b2-8789-f89850b5f3f1)
<--
write code in ci.yaml

__________________________________________________________________________________________________
```bash

name: CI

# Exclude the workflow to run on changes to the helm chart
on:
    push:
      branches:
        - main
      paths-ignore:
        - 'helm/**'
        - 'k8s/**'
        - 'README.md'
  
jobs:
  
    build:
      runs-on: ubuntu-latest
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v4
  
      - name: Set up Go 1.22
        uses: actions/setup-go@v2
        with:
          go-version: 1.22
  
      - name: Build
        run: go build -o go-web-app
  
      - name: Test
        run: go test ./...
    
    code-quality:
      runs-on: ubuntu-latest
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v4
  
      - name: Run golangci-lint
        uses: golangci/golangci-lint-action@v6
        with:
          version: v1.56.2
    
    push:
      runs-on: ubuntu-latest
  
      needs: build
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v4
  
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
  
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
  
      - name: Build and Push action
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}
  
    update-newtag-in-helm-chart:
      runs-on: ubuntu-latest
  
      needs: push
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.TOKEN }}
  
      - name: Update tag in Helm chart
        run: |
          sed -i 's/tag: .*/tag: "${{github.run_id}}"/' helm/go-web-app-chart/values.yaml
  
      - name: Commit and push changes
        run: |
          git config --global user.email "kavindugunasekara2000@gmail.com"
          git config --global user.name "WAKD Gunasekara"
          git add helm/go-web-app-chart/values.yaml
          git commit -m "Update tag in Helm chart"
          git push
```
__________________________________________________________________________________________________

3.Go to github repo
 in the repo go to settings
 go to secrets ans variable
 create New repository secret
       for 1.DOCKERHUB_USERNAME
           2.DOCKERHUB_PASSWORD
           3.DOCKERHUB_TOKEN

     ![30 1](https://github.com/user-attachments/assets/394d40e6-b727-446f-ad3e-d3709535d198)

FOR DOCKERHUB_TOKEN go to Docker hub 
 login to dockerhub
 go to account settong
 click personal access token
 generate new token
 
![31 1 1](https://github.com/user-attachments/assets/11cbf645-d441-460b-9559-95b3253c47e5)
_______________________________________________________________________________________________

![31 1](https://github.com/user-attachments/assets/4cf0ec65-0956-49b0-8e78-836a5f4be6d2)

final output of secret key 

![31 1 1 1](https://github.com/user-attachments/assets/ad6dfa4d-6eac-4b6b-bea1-307bc85e5ffe)


 This is importand  " tags: ${{ secrets.DOCKERHUB_USERNAME }}/go-web-app:${{github.run_id}}"
 so every commit the developer creates a new docker image is created and the tag for the new 
 docker image basically GitHub.runid , so every commit is unique and similarly that tag is create 
 for every commit if developer is creating 100 commits , then 100 commits will create 100 docker
 images , and they are pushed to the docker registry with GitHub.runid 

 we will do commit and check that docker image is created with new run id.

 Last stage 

 **Update the Helm**
 onece the new image is created with the new tag then we need to update the **HELM chart**
 
 to this we add that code to Ci.yaml
 ```bash
update-newtag-in-helm-chart:
      runs-on: ubuntu-latest
  
      needs: push
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.TOKEN }}
  
      - name: Update tag in Helm chart
        run: |
          sed -i 's/tag: .*/tag: "${{github.run_id}}"/' helm/go-web-app-chart/values.yaml
  
      - name: Commit and push changes
        run: |
          git config --global user.email "kavindugunasekara2000@gmail.com"
          git config --global user.name "WAKD Gunasekara"
          git add helm/go-web-app-chart/values.yaml
          git commit -m "Update tag in Helm chart"
          git push
```
completed the ci 

to commit that to github repository we need Github token
now start github 
  in the repo go to settings
  go to secrets ans variable
  create New repository secret 
  create new token as TOKEN 

  ![31 1 1 1](https://github.com/user-attachments/assets/1005a7e3-4ee9-46f1-ac0a-ac6522876e9f)

for this you need github action personal TOKEN 
 HOW TO GET IT ---

 go to main setttings
 go down of page
 click Developer settings
 personal access token
  tokens(classic) 
    crete the token  generate new token(classic)
  
  
 ![34 1](https://github.com/user-attachments/assets/2611c59e-8e47-44f8-b05a-934054ec91c6)

type golang-demo-cicd

![33 1](https://github.com/user-attachments/assets/6a7562a2-caf1-473d-8bbf-67a2479c4b89)

give permissions 

![33 1 1 1](https://github.com/user-attachments/assets/980e7631-4fa6-4950-b8b7-ecc2399913b2)

generate token

copy and past it on secret access token section password

 **END THE CI PART**
-----------------------------------------------------------------------------------------------

**CHECK CI IS WORKING FINE OR NOT**

START ADD THIS FILES TO GITHUB

```bash
 git add .
 git commit -am " implementrd ci"
 git push

 ```
go to repo 



Now pipeline is running

 ![33 2 2](https://github.com/user-attachments/assets/15580af3-6a42-4ae9-aaa0-e34236715a09)

 Go to actions section
  
![28create ci pipeline](https://github.com/user-attachments/assets/218dc98b-8ce8-4640-8368-1ad0f9d39f95)

after RUN the Ci successfull ---->

Now docker images getting created

go to docker hub

go to go-web-app

go to the TAGS
let's see new tag has new tag has push to docker hub

![new tag with runner id](https://github.com/user-attachments/assets/cc883ddd-4d72-43f3-97d5-89680d9f587d)

Now go to repo helm folder 
go to the values.yaml


**__________________you will see new tag has updated ___________________**

![30successfull update docker tag in helm](https://github.com/user-attachments/assets/f9181b2a-5346-494c-a801-1eaa2e92871b)

****Now CI has created sucessfully** **

Now only have Argocd part for Cd

now it is noly left with the Argocd every time the ci pipline is run Argocd has to identify the 
change and push it to the k8s cluster.


**Lets start implement ArgoCd part**

Install Argo CD
 Install Argo CD using manifests
 ```bash 
 kubectl create namespace argocd
 kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo- 
 cd/stable/manifests/install.yaml
```
In your k8s cluster  argocd will be install 

lets argocd will install compled
![31install ARGOCD](https://github.com/user-attachments/assets/946a9840-edc4-43b2-92f2-1b6e9cbfb5d3)

 Access the Argo CD UI (Loadbalancer service)
  ```bash 
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
Access the Argo CD UI (Loadbalancer service) -For Windows

  ```bash 
kubectl patch svc argocd-server -n argocd -p '{\"spec\": {\"type\": \"LoadBalancer\"}}'
```
![32ssAccess the Argo CD UI (Loadbalancer service)](https://github.com/user-attachments/assets/0a50f2f6-e598-408a-a044-cf0d9128d862)


--------------------------------------------------

check argocd exteranl ip
  ```bash 
kubectl get svc -n argocd

```

acees to argocd

```bash
kubectl get nodes -o wide
```

![34try to access ARGOCD](https://github.com/user-attachments/assets/ebda643a-ee78-4fa8-a4b6-15f73d60183c)

get tha ip and node port to accees using it access the argocd

54.160.199.111:32434

![35 1](https://github.com/user-attachments/assets/240730e6-9c77-4c07-aa49-87f9a85276d4)

you need sign in
username admin


to get password run this commands
```bash
kubectl get secrets -n argocd
kubectl edit secret argocd-install-admin-secret -n argocd

```

![36 1](https://github.com/user-attachments/assets/515b62b4-d993-4080-a4cc-d706a11a1e98)

password decode 


![36 1 1](https://github.com/user-attachments/assets/d8ff6c03-904c-4b9d-9cca-eef0207ef6fa)

--------------------------------------------------------------------------------------

In windows use this to get argocd password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | 
    ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) };
```
![38 argocd password get](https://github.com/user-attachments/assets/cb91cf51-31b3-49f6-9b80-a2630630c06b)


using password sign in to argocd

successfully sign in to


argocd is on same k8s cluster 

in argocd dash board 
![37 1](https://github.com/user-attachments/assets/aa951000-5d24-4c90-b258-1175f310af0c)

cleick new app

give name as go-web-app

sync policy Automatic

click self heal

repository url 
put right link here

then it auto matically identofy the folder

![38 1](https://github.com/user-attachments/assets/93f44975-1198-49af-b866-f30eee1ff98a)

select  values.yaml

![38 1 1](https://github.com/user-attachments/assets/8289be8c-6382-4faa-bded-5f1cf03850e3)


cleck **create**


argocd will look all the files in your github repository with in the helm chart it will update
the values.yaml with all the changes that are required in your deployment and all

**now you will see argocd has started deploying evrythig you see everything is created pod ,svc ,
ingreess in created**

![39Argocd work successfully](https://github.com/user-attachments/assets/58e4e0be-be39-49eb-be46-2be03b6c29fe)


check every thing created successfully

![40 successfull](https://github.com/user-attachments/assets/0757e2e8-3d24-4e5e-8643-b8a0b2c1c4e0)

Now access the application

![41success](https://github.com/user-attachments/assets/188c207d-01d5-4790-9f34-ea9709873b5a)

**Using CICD now application has deployed ---- Access application**

https://github.com/user-attachments/assets/f0b5b1ed-207f-4130-9f80-7c03eabb4364

**To verify CICD is work fine lets make some changes**

make changes on home,html in github then commit it
go to home.html then add some text on page what ever you want 
then commit the changes 


lets ci will run 
![43start ci work after change](https://github.com/user-attachments/assets/b7f42bdf-a1e3-4fc1-a9ab-63a3b0222eaf)


then new docker images should be created

https://github.com/user-attachments/assets/fe4d88f3-0e12-4bd9-85e4-00fa5ea70ba8

And new docker image should be tagged  it shloud be pushed to the docker hub
ones it succesfull we have new tag

![45 after new chabge tag has changed](https://github.com/user-attachments/assets/b78ab55a-aa51-42cb-8291-5fa4dd3f0ab6)

Valus.yaml also updated ,
Argocd has to pickup this new change, argocd by default looks for new changes  ,
that's why even if you goes to the k8s cluster execute

```bash
kubectl get deploy
kubectl edit deploy go-web-app

```
you will see images has updated already

![47tag change argocd](https://github.com/user-attachments/assets/0b313a1d-faa2-4f01-950c-a57bf3384669)

to verify we can refresh the applcaiton form browser then you will see we apply update on 
home.html appear on it , successfully apply new changes 
--------------------------------------------------------------------------------------------
 **Let's see how new changes will apply**


https://github.com/user-attachments/assets/f381b7d8-62d9-4a0e-b6e5-8a6dfac3533b


**Final work what we done in this project**

![finalDevopsified ](https://github.com/user-attachments/assets/c83be1c7-4c0d-4f47-ba00-c700e95bb6fd)

1. Containerization with multistage docker file (implement images with reduce images as well as security)
2. K8s manifest such as deplyment ,serivce , ingress
3. Setup Ci using Github Action
4. Cd using GitOps ( using ArgoCd to implement continous delivery)
5. k8 cluster set up beacuse ci cd has to deploy applicaion on the target platform then it is K8s(then application deploy on EKS cluster)
6. set up helm chart Because that use full when dev team need to deploy application on Develop, QA or Production and muliple environment instend of writing manifest for each environment then they can use helm chart pass the values.yaml

7. set ingress controller configuration so that would create loadbalancer depending upon ingress configuration -> so that application expose to the outside world

8. how to map load balancer ip address to loacl DNS so that we test if the application is access from the outside world
