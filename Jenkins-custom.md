## Get Jenkins to work with https

### solved with jenkins docker container by converting everything to a keystore

### first added a password to my key

### Note for keytool to work you need to make sure you install Java

````
sudo apt install default-jre

````

````
openssl rsa -des3 -in key.pem -out key.encrypted.pem

then converted to pkcs12
openssl pkcs12 -inkey key.encrypted.pem -in cert.pem -export -out keys.encrypted.pkcs12

then created a keystore (password for keystore should be same as password for key)
keytool -importkeystore -srckeystore keys.encrypted.pkcs12 -srcstoretype pkcs12 -destkeystore keystore

then updated Dockerfile to include keystore and a reference to it in JENKINS_OPTS
{{FROM jenkins
USER root
RUN apt-get update && apt-get install -y jq
USER jenkins
COPY keystore /var/lib/jenkins/keystore
ENV JENKINS_OPTS --httpPort=8080 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/keystore --httpsKeyStorePassword=whateverpasswordyouspecified
EXPOSE 8443}}
````

[jenkins.io](https://issues.jenkins.io/browse/JENKINS-22448)


````
FROM jenkins/jenkins:lts-jdk11

COPY --chown=jenkins:jenkins keystore /var/lib/jenkins/keystore
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/keystore --httpsKeyStorePassword=keystore_password_goes_here
EXPOSE 8443
````


## Runnig Jenkins as a docker container has to be setup in a special way so it can run docker commands inside of itself DinD

````
docker network create jenkins

docker volume create jenkins-data

docker volume create jenkins-docker-certs
````

````

docker run --name jenkins-docker -d \
--privileged --network jenkins --network-alias docker \
--env DOCKER_TLS_CERTDIR=/certs \
--volume jenkins-docker-certs:/certs/client \
--volume jenkins-data:/var/jenkins_home \
--publish 2376:2376 \
--restart=always \
docker:dind --storage-driver overlay2
````

## Now our custom Jenkins image


````
FROM jenkins/jenkins:lts-jdk11

USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli

USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.2 docker-workflow:1.27"
COPY --chown=jenkins:jenkins keystore /var/lib/jenkins/keystore
ENV JENKINS_OPTS --httpPort=-1 --httpsPort=8443 --httpsKeyStore=/var/lib/jenkins/keystore --httpsKeyStorePassword=kuN>d2sw
EXPOSE 8443

````

### Now let's run our custom image

````
docker run --name jenkins-blueocean -d \
--network jenkins --env DOCKER_HOST=tcp://docker:2376 \
--env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
--env JAVA_OPTS=-Dhudson.footerURL=https://jenkins.fbclouddemo.us \
--publish 443:8443 --publish 50000:50000 \
--volume jenkins-data:/var/jenkins_home \
--volume jenkins-docker-certs:/certs/client:ro \
--restart=always \
local/jenkins-blueocean:lts
````
