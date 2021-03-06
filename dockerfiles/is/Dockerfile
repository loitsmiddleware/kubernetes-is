# ------------------------------------------------------------------------
#
# Copyright 2017 WSO2, Inc. (http://wso2.com)
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License
#
# ------------------------------------------------------------------------

FROM ubuntu:16.04
MAINTAINER WSO2 Docker Maintainers "dev@wso2.org"
ENV DEBIAN_FRONTEND noninteractive

# set user information
ARG USER=wso2user
ARG USER_GROUP=wso2
ARG USER_ID=1000000000
ARG USER_HOME=/home/${USER}

# Set startup script
ARG STARTUP_SCRIPT=dockerfiles/is/init_carbon.sh

# Set startup script - inside 
ARG STARTUP_SCRIPT_IN=init_carbon.sh

# set local files directory
ARG FILES=dockerfiles/is/files

# set jdk information
ARG JDK_ARCHIVE=jdk-8u*-linux-x64.tar.gz
ARG JAVA_HOME=${USER_HOME}/java

# set wso2 product information
ARG WSO2_SERVER=wso2is
ARG WSO2_SERVER_VERSION=5.5.0
ARG WSO2_SERVER_PACK=${WSO2_SERVER}-${WSO2_SERVER_VERSION}.zip
ARG WSO2_SERVER_HOME=${USER_HOME}/${WSO2_SERVER}-${WSO2_SERVER_VERSION}

# set jdbc driver information
ARG JDBC_DRIVER=mysql-connector-java-5.1.31-bin.jar

# set kubernetes dependency bundle information
ARG KUBERNETES_MEMBERSHIP_SCHEME_LIB=kubernetes-membership-scheme-1.0.*.jar
ARG DNS_JAVA_LIB=dnsjava-2.1.8.jar

# install required packages
RUN apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
	   wget \
       	   zip \
           unzip \
           telnet \
           iproute2 \
	   software-properties-common python-software-properties \
	   sudo && rm -rf /var/lib/apt/lists/*

# create user group and user
RUN useradd --system --uid ${USER_ID} --gid 0 --create-home --home-dir ${USER_HOME} \
	--no-log-init ${USER}


# Install Java.
RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer

RUN wget --user-agent="testuser" --referer="http://connect.wso2.com/wso2/getform/reg/new_product_download"  http://product-dist.wso2.com/products/identity-server/5.5.0/wso2is-5.5.0.zip -P /${USER_HOME} 

COPY ${STARTUP_SCRIPT} ${USER_HOME}/scripts/init_carbon.sh

# copy jdk, wso2 product distribution, jdbc driver and kubernetes dependency libraries to user's home directory
COPY ${FILES}/${JDK_ARCHIVE} ${STARTUP_SCRIPT} ${FILES}/${JDBC_DRIVER} \
     ${FILES}/${DNS_JAVA_LIB} ${FILES}/${KUBERNETES_MEMBERSHIP_SCHEME_LIB}  dockerfiles/is/sudoers dockerfiles/is/change_ownership.sh ${USER_HOME}/
     
# install jdk, wso2 server, remove distributions and set folder permissions
RUN unzip -q ${USER_HOME}/${WSO2_SERVER_PACK} -d ${USER_HOME}/ \
    && mkdir -p ${USER_HOME}/tmp \
    && cp -r ${USER_HOME}/${WSO2_SERVER}-${WSO2_SERVER_VERSION}/repository/deployment/server ${USER_HOME}/tmp/ \
    && mkdir -p ${JAVA_HOME} \
    && cp ${USER_HOME}/${JDBC_DRIVER} ${USER_HOME}/${DNS_JAVA_LIB} ${WSO2_SERVER_HOME}/repository/components/lib/ \
    && cp ${USER_HOME}/${KUBERNETES_MEMBERSHIP_SCHEME_LIB} ${WSO2_SERVER_HOME}/repository/components/dropins/ \
    && mkdir -p ${USER_HOME}/scripts/ \
    && cat ${USER_HOME}/${STARTUP_SCRIPT_IN} > ${USER_HOME}/scripts/${STARTUP_SCRIPT_IN} \
    && rm ${USER_HOME}/${STARTUP_SCRIPT_IN} \
    && rm ${USER_HOME}/${WSO2_SERVER_PACK} \
    #&& rm ${USER_HOME}/${JDK_ARCHIVE} \
    && rm ${USER_HOME}/${JDBC_DRIVER} \
    && rm ${USER_HOME}/${DNS_JAVA_LIB} \
    && rm ${USER_HOME}/${KUBERNETES_MEMBERSHIP_SCHEME_LIB} \
    && chgrp -R 0 ${USER_HOME} \
    && chmod -R g=u ${USER_HOME} \
    && chown -R ${USER_ID} ${USER_HOME} \
    && chmod -R 0774 ${USER_HOME} \
	&& cat ${USER_HOME}/sudoers > /etc/sudoers \
	&& rm ${USER_HOME}/sudoers \
	&& cat ${USER_HOME}/change_ownership.sh > /bin/change_ownership.sh \
	&& chmod 0755 /bin/change_ownership.sh \
	&& rm ${USER_HOME}/change_ownership.sh \
	&& chgrp -R 0 ${USER_HOME} \
	&& chmod -R g=u ${USER_HOME} \
	&& chown -R ${USER_ID} ${USER_HOME} \
	&& chmod -R 0774 ${USER_HOME}

# set the user and work directory
USER ${USER_ID}
WORKDIR ${USER_HOME}

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

# set environment variables
ENV JAVA_HOME=${JAVA_HOME} \
    PATH=$JAVA_HOME/bin:$PATH \
    USER_HOME=${USER_HOME} \
    WSO2_SERVER=${WSO2_SERVER} \
    WSO2_SERVER_VERSION=${WSO2_SERVER_VERSION} \
    WSO2_SERVER_HOME=${WSO2_SERVER_HOME} \
    USER=${USER}

# expose ports
EXPOSE 9763 9443

ENTRYPOINT exec ${USER_HOME}/scripts/init_carbon.sh
