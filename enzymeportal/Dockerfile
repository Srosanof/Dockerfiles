FROM centos:latest


#Get common libraries
RUN yum -y install wget git unzip gcc vim which

#Install Java JDK 1.8.0.60
RUN wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.rpm" \
    && rpm -ivh jdk-8u60-linux-x64.rpm \
    && rm jdk-8u60-linux-x64.rpm \
    && echo 'export JAVA_HOME=/usr/java/jdk1.8.0_60' >> /root/.bashrc \
    && echo 'export JAVA_PATH=/usr/java/jdk1.8.0_60' >> /root/.bashrc

#Install maven
RUN mkdir maven \
    && cd /maven \
    && wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz \
    && tar xzvf /maven/apache-maven-3.3.9-bin.tar.gz \
    && rm /maven/apache-maven-3.3.9-bin.tar.gz \
    && echo 'export M2_HOME=/maven/apache-maven-3.3.9' >> /root/.bashrc

   
#Set Environment Variables for Docker Build
ENV M2_HOME /maven/apache-maven-3.3.9
ENV JAVA_HOME /usr/java/jdk1.8.0_60
ENV JAVA_PATH /usr/java/jdk1.8.0_60
ENV USER root
ENV PATH $PATH:${JAVA_HOME}/bin:${M2_HOME}/bin

#Set PATH on container startup
RUN echo 'export PATH=$PATH:${JAVA_HOME}/bin:${M2_HOME}/bin' >> /root/.bashrc

#Install tomcat 8 and create tomcat user
RUN  groupadd tomcat \
     && mkdir opt/tomcat \
     && useradd -s /bin/nologin -g tomcat -d /opt/tomcat tomcat \
     && wget http://www-us.apache.org/dist/tomcat/tomcat-8/v8.0.43/bin/apache-tomcat-8.0.43.tar.gz \
     && tar -zxvf apache-tomcat-8.0.43.tar.gz -C /opt/tomcat --strip-components=1 \
     && rm apache-tomcat-8.0.43.tar.gz \
     && cd /opt/tomcat \
     && chgrp -R tomcat conf \
     && chmod g+rwx conf \
     && chmod g+r conf/* \
     && chown -R tomcat logs/ temp/ webapps/ work/ \
     && chgrp -R tomcat bin \
     && chmod g+rwx bin \
     && chmod g+r bin/* 
 
#Checkout Taxonomy code from GitHub 
RUN mkdir git-repository \ 
    && cd git-repository \
    && git clone -b docker https://github.com/uniprot/enzymeportal.git \
    && cd /git-repository/enzymeportal \
    && chmod -Rf 755 *


#Maven install oracle jdbc
RUN cd /git-repository/enzymeportal/enzyme-portal \
    && ${M2_HOME}/bin/mvn install -DskipTests
    

#Expose ports for service
EXPOSE 8080 8097 8089

#CMD ./enzymeportal/bin/start.sh
