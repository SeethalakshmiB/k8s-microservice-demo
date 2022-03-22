FROM centos:latest

RUN mkdir /opt/tomcat/

WORKDIR /opt/tomcat/
RUN curl -O https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.60/bin/apache-tomcat-9.0.60.tar.gz
RUN tar xvzf apache-tomcat-*.tar.gz
RUN mv apache-tomcat-*/* /opt/tomcat/.
RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum update -y && yum install java -y
RUN java -version

WORKDIR /opt/tomcat/webapps
ADD helloworld-jsp.war .
ADD sql-demo.war .

EXPOSE 8080

CMD ["/opt/tomcat/bin/catalina.sh", "run"]