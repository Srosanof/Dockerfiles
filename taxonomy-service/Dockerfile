FROM centos:latest


#Get common libraries
RUN yum -y install wget rsync git unzip which libaio

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

   
#Install Oracle full client
 ADD oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm /oracle/
 ADD oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm /oracle/
 ADD oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm /oracle/
 RUN rpm -ivh /oracle/oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm 
 RUN rpm -ivh /oracle/oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm 
 RUN rpm -ivh /oracle/oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm 
 RUN rm -rf /oracle/ \
     && ln -s /usr/lib/oracle/11.2/client64 /oracle \
     && rm /usr/bin/sqlplus64 \
     && cd /usr/bin && ln -s /usr/lib/oracle/11.2/client64/bin/sqlplus sqlplus \
     && echo 'export ORACLE_HOME=/usr/lib/oracle/11.2/client64' >> /root/.bashrc

#Setup monitoring tool jar
RUN mkdir monitoring-tool \
    && mkdir monitoring-tool/dist \
    && wget -O jmx_prometheus_javaagent-0.7.jar  http://search.maven.org/remotecontent?filepath=io/prometheus/jmx/jmx_prometheus_javaagent/0.7/jmx_prometheus_javaagent-0.7.jar  \
    && mv jmx_prometheus_javaagent-0.7.jar /monitoring-tool/dist/jmx_prometheus_javaagent-0.7-SNAPSHOT.jar



#Set Environment Variables for Docker Build
ENV ORACLE_HOME /usr/lib/oracle/11.2/client64
ENV M2_HOME /maven/apache-maven-3.3.9
ENV JAVA_HOME /usr/java/jdk1.8.0_60
ENV JAVA_PATH /usr/java/jdk1.8.0_60
ENV USER root
ENV PATH $PATH:${JAVA_HOME}/bin:${M2_HOME}/bin:${ORACLE_HOME}/bin

#Set PATH on container startup
RUN echo 'export PATH=$PATH:${JAVA_HOME}/bin:${M2_HOME}/bin:${ORACLE_HOME}/bin' >> /root/.bashrc

 
#Checkout Taxonomy code from GitHub 
RUN mkdir taxonomy \
    && mkdir taxonomy/git-repository \ 
    && cd /taxonomy/git-repository \
    && git clone -b docker https://github.com/ebi-uniprot/unp.fw.taxonomy.git \
    && cd /taxonomy \
    && cp -rf /taxonomy/git-repository/unp.fw.taxonomy/taxonomy-deployment/src/main/resources/* . \
    && chmod 755 *.sh \
    && cd /taxonomy/bin \
    && chmod 755 *.*


#Maven install oracle jdbc
RUN ${M2_HOME}/bin/mvn install:install-file -Dfile=${ORACLE_HOME}/lib/ojdbc6.jar -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.4 -Dpackaging=jar
    

#Remove Oracle clients from image
RUN rm -rf /usr/lib/oracle \
    && rm -rf /oracle

#Build taxonomy
RUN cd /taxonomy/bin \
    && ./create-taxonomy-release-files.sh 2017_02


#Adding and Extract Latest Release Neo4j database
#ADD neo4j-taxonomy-database.tar.gz /taxonomy/releases/2017_02/

RUN cd /taxonomy/releases/2017_02 \
    && wget http://ftp.ebi.ac.uk/pub/contrib/UniProtKB/taxonomy/neo4j-taxonomy-database.tar.gz \
    && tar xzvf neo4j-taxonomy-database.tar.gz 
RUN rm -rf neo4j-taxonomy-database.tar.gz

    
RUN cd /taxonomy \
    && ln -s /taxonomy/releases/2017_02 currentrelease \
    && touch /taxonomy/releases/2017_02/logs/application.log \
    && touch /taxonomy/releases/2017_02/logs/console.log 
    
    
#Expose ports for service
EXPOSE 9090 9091 9096 9991

CMD ./taxonomy/bin/start.sh
