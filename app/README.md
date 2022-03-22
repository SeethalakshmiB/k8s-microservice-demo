# K8S application - Micro service
Database <--> Tomcat <--> Apache <-- user

Tomcat <--> Apache 

## Apache Pods

### Build Apache image from Dockerfile
    - create custom image --> 
        - $ docker build -t k8s-demo-apache .
        - $ docker images
    - Push custom image into Docker hub 
        -  $ docker image tag k8s-demo-apache seetha123/k8s-demo-apache
        -  $ docker push seetha123/k8s-demo-apache 

### Create apache pod
  - kubectl apply -f apache-pod.yaml 

## Tomcat Pods

### Build tomcat image from Dockerfile
    - create custom image --> 
        - $ docker build -t k8s-demo-tomcat .
        - $ docker images
    - Push custom image into Docker hub 
        -  $ docker image tag k8s-demo-tomcat seetha123/k8s-demo-tomcat
        -  $ docker push seetha123/k8s-demo-tomcat

### Create apache pod
  - kubectl apply -f apache-pod.yaml 

## Make services for apache and tomcat
  
### Apache service
- kubectl apply -f apache-svc.yaml

### Tomcat Service
- kubectl apply -f tomcat-svc.yaml

manual work>
### Login to apache pod 
    - edit /etc/httpd/conf/httpd.conf

<VirtualHost *:80>
    # ServerName maps.example.com
    # forward apache 80 to tomcat 8080
    ProxyPass / http://tomcat-svc:8080/
    ProxyPassReverse / http://tomcat-svc:8080/
</VirtualHost>