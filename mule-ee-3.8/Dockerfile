###############################################################################
## Dockerizing Mule EE
## Version:  2.0
# Based on example provided by Ken Ng and Nial Darby
# Updates include:
#  - From Ubuntu 16:04
#  - Add current Oracle 8 JDK to image
#  - Add current ARM Agent 1.7.0

## Update arg values based on version of Mule and Mule ARM Agent version
## not that you'll need to provide the corresponding zip files for the Docker build
###############################################################################

FROM                    ubuntu:16.04
MAINTAINER              Amir Kader <amir@mulesoft.com>

###############################################################################
## Setting up the arguments
## Update arg values based on version of Mule and Mule ARM Agent version
## not that you'll need to provide the corresponding zip files for the Docker build

ARG muleVersion=3.8.4
ARG muleAgentVersion=1.7.0
ARG muleHome=/opt/mule/mule-enterprise-standalone-${muleVersion}
ARG muleDistribution=mule-ee-distribution-standalone-${muleVersion}.zip

###############################################################################
## Base container configurations

RUN apt-get update \
	&& apt-get upgrade -y \
	&& apt-get install -y  apt-utils \
	&& apt-get install -y software-properties-common \
	&& add-apt-repository ppa:webupd8team/java -y \
	&& apt-get update \
	&& echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | /usr/bin/debconf-set-selections \
	&& apt-get install -y oracle-java8-installer \
	&& apt-get install -y zip unzip \
	&& apt-get install -y curl \
	&& apt-get install -y jq \
	&& apt-get clean \

###############################################################################
## MuleEE installation:

## Install Mule EE
WORKDIR /opt/mule
COPY ./agent-setup-${muleAgentVersion}.zip /opt/mule/
COPY ./${muleDistribution} /opt/mule/
RUN ls -al \
	&& cd /opt/mule/ \
	&& unzip ${muleDistribution} \
	&& ls -al \
	&& unzip -o agent-setup-${muleAgentVersion}.zip -d ${muleHome}/bin \
	&& rm ${muleDistribution} \
	&& rm agent-setup-${muleAgentVersion}.zip
ADD ./startMule.sh ${muleHome}/bin/
RUN chmod 755 ${muleHome}/bin/startMule.sh

## [OPTIONAL] Copy the Datamapper plugin
# ADD     ./data-mapper-plugin-$muleVersion.zip /opt/mule/plugins/

## Copy the License file - pre-package into docker image to avoid leakage
#COPY ./license.lic /opt/mule/
#RUN /${muleHome}/bin/mule -installLicense /opt/mule/license.lic

###############################################################################
## Configure mule runtime access pre-requisites

## HTTPS Port for Anypoint Platform communication
EXPOSE  443

## Mule remote debugger
EXPOSE  5000

## Mule JMX port (must match Mule config file)
EXPOSE  1098

## Mule Cluster ports
EXPOSE 5701
EXPOSE 54327

###############################################################################
## Expose the necessary port ranges as required by the apps to be deployed

## HTTP Service Port
EXPOSE 8081

## HTTPS Service Port
EXPOSE 8091

###############################################################################

## Environment and execution:
ENV MULE_BASE /opt/mule/mule-enterprise-standalone-${muleVersion}
WORKDIR  ${MULE_BASE}/bin

#In order to access environment variables and arguments passed from the startDockerContainer script to the startMule.sh
#the ENTRYPOINT needed to be updated
ENTRYPOINT ["/bin/bash", "-c",  "$MULE_BASE/bin/startMule.sh ${*}", "--" ]
