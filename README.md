# SonarQube Docker Compose

This is a quick set up to install SonarQube and its associated technology. The main bulk of the installation are based on Docker containers.

## Technolog Stack

1. SonarQube Community Edition 7.8
2. Postgresql 10
3. Nginx
4. OpenJDK 8

## Getting Started

Assuming your systems are not connected the internet or required interaction with internal systems that are **secured by internal certificate authority**, you may need to build the respective docker images locally first on an internet machine.

## Pre-requisite

Please ensure these tools are installed in your system.

1. docker
2. docker-compose

### Build the Docker Image

1. Build Postgresql Image

```
cd /build/postgresql
docker build --tag postgresql:10 --file Dockerfile .

# Save the image as tar
docker save postgresql:10  > postgresql-10.tar
```

2. Build SonarQube Image

The Dockerfile for SonarQube included stepps to load self-signed certificates into the container's java truststore. This is important if you required interaction to internal systems that enabled TLS with self-signed certificates. Copy the needed certificates based on your context.

```
# Example for copying a root ca to java trust store

COPY /certs/root.crt /tmp/

RUN keytool -import -v -trustcacerts -alias sonarqube -file /tmp/root.crt \
    -keystore ${JAVA_HOME}/jre/lib/security/cacerts -noprompt -storepass changeit
```

```
cd /build/sonarqube
docker build --tag sonarqube:hive --file Dockerfile .

# Save the image as tar
docker save sonarqube:hive  > sonarqube-hive.tar
```

### Copy The Docker Container Images

Copy both the tar files into your machine or vm.

### Running Docker Compose

Clone or copy this repository into the machine

```
cd docker-compose-sonarqube

docker-compose up -d
```

SonarQube should be up and running. Try to access it by going to http://localhost:9000

## Editing the docker-compose file

### Specifying your postgresql parameters

Edit the env variables based on your own set-up. Note : This is not a secure or recommended way of managing secrets in containers, please do check out solutions like Hashicorp's Vault if you intend to run this in enterprise level.

```
   environment:
      - POSTGRES_USER=xxxx
      - POSTGRES_PASSWORD=xxxx
      - POSTGRES_DB=xxx
```

## Editing SonarQube Configuration

You can also edit the configuration at conf/sonar.properties as it is mount directly to the container. The property file has been sanitized without actual secrets , ips or passwords.

The example configurations in sonar.properties to connect with AD through ldaps and outgoing proxy for updates through HTTP authentication. Please add in the details based on your own set-up

##

## License

[Robin Cher - MIT LICENSE ](https://github.com/robincher/docker-compose-sonarqube/blob/master/LICENSE)

## Useful Commands

Some useful commands for testing the set-up

### LDAP Binding Test

```
ldapwhoami -vvvv -H ldaps://ldapserver  -D "service_account@domain" -W
```

## Reference

1. Adding TrustStore - https://github.com/SonarSource/docker-sonarqube/issues/207
