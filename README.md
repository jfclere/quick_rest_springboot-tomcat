# Introduction

This project exposes a simple REST endpoint where the service `greeting` is available at this address `http://hostname:port/greeting` and returns a json Greeting message

```
{
    "content": "Hello, World!",
    "id": 1
}

```

The id of the message is incremented for each request. To customize the message, you can pass as parameter the name of the person that you want to send your greeting.

# Build

The project bundles the Apache Tomcat 8.0.36 artifacts with SpringBoot 1.4.1.RELEASE.

To build the project, use this maven command.

```
mvn clean install
```

# Launch and test

To start Spring Boot , run the following commands in order to start the maven goal of Spring Boot

```
mvn spring-boot:run
```

If the application has been launched without any error, you can access the REST endpoint exposed using curl or httpie tool

```
http http://localhost:8080/greeting
curl http://localhost:8080/greeting
```

To pass a parameter for the Greeting Service, use this HTTP request

```
http http://localhost:8080/greeting name==Charles
curl http://localhost:8080/greeting -d name=Bruno
```

# OpenShift

The Project can be deployed top of Openshift using the [minishift tool](https://github.com/minishift/minishift) who will take care to install within a Virtual machine (Virtualbox, libvirt or Xhyve) the OpenShift platform
like also a Docker daemon. For that purpose, you will first issue within a terminal the following commands.

```
minishift delete
minishift start --openshift-version=v1.3.1
eval $(minishift docker-env)
oc login --username=admin --password=admin
```

Next, we will use the Fabric8 Maven plugin which is a Java OpenShift/Kubernetes API able to communicate with the prlatform in order to request to build the docker image and next to create using Kubernetes
a pod from the image of our application.

A maven profile has been defined within this project to configure the Fabric8 Maven plugin

```
mvn clean fabric8:build -Popenshift
```

Remark : To use the official Red Hat S2I image, then we must configure the Fabric8 Maven Plugin to use the Java S2I image with this parameter `-Dfabric8.generator.from=registry.access.redhat.com/jboss-fuse-6/fis-java-openshift`

Next we can deploy the templates top of OpenShift and wait till kubernetes has created the POD

```
mvn fabric8:deploy -Popenshift
```

Then, you can test the service deployed in OpenShift and get a response message 

```
http $(minishift service NAME_OF_THE_SERVICE --url=true)/greeting
```

To test the project against OpenShift using Arquillian, simply run this command

```
mvn clean verify -Popenshift
```

# OpenShift Online

- Connect to the OpenShift Online machine (e.g. https://console.dev-preview-int.openshift.com/console/command-line) to get the token to be used by the oc client to be authenticated and access the project
- Open a terminal and execute this command using the oc client where you will replace the MYTOKEN with the one that you can get from the Web Console
```
oc login https://api.dev-preview-int.openshift.com --token=MYTOKEN
```
- Use the Fabric8 Maven Plugin to launch the S2I process on the OpenShift Online machine
```
mvn clean fabric8:deploy -Popenshift -DskipTests
```
- And to run/launch the pod
```
mvn fabric8:start -Popenshift -DskipTests
```
- Create the route to access the service 
```
oc expose service/NAME_OF_THE_SERVICE --port=8080 
```
- Get the route url
```
oc get route/ROUTE_NAME
NAME         HOST/PORT                                                    PATH      SERVICE           TERMINATION   LABELS
demo   demo-obsidian.1ec1.dev-preview-int.openshiftapps.com             demo:8080                 expose=true,group=org.jboss.quickstart,project=springboot-rest,provider=fabric8,version=1.0-SNAPSHOT
```
- Use the Host/Port address to access the REST endpoint
```
http http://demo-obsidian.1ec1.dev-preview-int.openshiftapps.com/greeting
http http://demo-obsidian.1ec1.dev-preview-int.openshiftapps.com/greeting name==Bruno

or 

curl http://demo-obsidian.1ec1.dev-preview-int.openshiftapps.com/greeting
curl http://demo-obsidian.1ec1.dev-preview-int.openshiftapps.com/greeting -d name=Bruno

```
