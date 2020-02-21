FROM phusion/baseimage:0.11
ENV HOME /root
MAINTAINER MichaelLow <michael.low.82@gmail.com>

# Regenerate SSH host keys. baseimage-docker does not contain any, so you
# have to do that yourself. You may also comment out this instruction; the
# init system will auto-generate one during boot.
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64
RUN add-apt-repository ppa:openjdk-r/ppa 
RUN apt-get update
RUN apt-get install -y openjdk-8-jdk
RUN apt-get install wget
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN mkdir /photon
WORKDIR /photon

# Expose Photon Webservice & ES
EXPOSE 2322
EXPOSE 9200

# Download photon release
RUN rm -rf /photon/photon_data/elasticsearch/data
RUN wget https://github.com/komoot/photon/releases/download/0.3.2/photon-0.3.2.jar
RUN java -jar /photon/photon-0.3.2.jar  -nominatim-import -host host.docker.internal -port 5432 -languages en

ENTRYPOINT ["/usr/lib/jvm/java-8-openjdk-amd64/bin/java"]
CMD ["-jar", "/photon/photon-0.3.2.jar"]