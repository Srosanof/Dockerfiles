FROM httpd:latest

WORKDIR /usr

#Create additional users in container
RUN useradd --system --uid 71379 -m --shell /usr/sbin/nologin phantomjs \
    && useradd --system --uid 20317 -m --shell /usr/sbin/nologin quickgo

#Install libraries needed by node, npm, and phantomjs
RUN apt-get update \
&& apt-get install -y --force-yes --no-install-recommends \
ca-certificates \
libsqlite3-dev \
libfontconfig1-dev \
libicu-dev \
libfreetype6 \
libssl1.0.0=1.0.1t-1+deb8u6 \
libssl-dev \
openssl \
libpng-dev \
libjpeg-dev \
vim \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*

#Additional libraries needed during install but can be removed before creating final image
RUN apt-get update \
&& apt-get install -y --no-install-recommends \
build-essential \
curl \
g++ \
gcc \
git \
flex \
bison \
gperf \
make \
perl \
python \
ruby \
sudo \
wget

#Get the node installation
RUN curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash - \
    && apt-get install -y nodejs

#Install bower, grunt, phantomjs addons
RUN npm install -g bower \
    && npm install -g grunt \
    && npm install -g grunt-cli \
    && npm install -g phantomjs

#Checkout QuickgoFE code from github
RUN mkdir -p /usr/quickgo \
    && mkdir -p /usr/quickgo/git-repository \
    && cd /usr/quickgo/git-repository \
    && git clone https://github.com/ebi-uniprot/QuickGoFE \
    && cd /usr/quickgo/git-repository/QuickGoFE \
    && chmod 775 -R /usr/quickgo/git-repository/QuickGoFE \
    && npm install

#Bower install on checked out code
RUN cd /usr/quickgo/git-repository/QuickGoFE \
    && bower --allow-root install

#Remove obsolete libraries not needed in final image
RUN apt-get purge --auto-remove -y \
build-essential \
g++ \
gcc \
git \
flex \
bison \
gperf \
make \
ruby \
perl \
python \
&& apt-get clean \
&& rm -rf /tmp/* /var/lib/apt/lists/*

COPY conf.d /usr/local/apache2/conf/extra/
RUN echo "Include conf/extra/conf.d" >> /usr/local/apache2/conf/httpd.conf

ENV APACHE_HOME /usr/local/apache2

#Expose ports
EXPOSE 8080 8910
