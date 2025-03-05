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










   


