#Jenkins agent to build java 21, maven and git

#FROM jenkins/inbound-agent:jdk21
#
#USER root
#
#RUN apt-get update && apt-get install -y maven git && rm -rf /var/lib/apt/lists/*
#
#USER jenkins


#Jenkins agent to build Docker

FROM jenkins/inbound-agent:jdk21

USER root

RUN apt-get update && apt-get install -y docker.io && rm -rf /var/lib/apt/lists/*

USER jenkins