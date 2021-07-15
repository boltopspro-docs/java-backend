<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/java-backend/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# Demo Java Backend App

Simple java project demos how to build a war file to be deployed on a Tomcat server.

## Dependencies

* docker: `brew install docker`
* maven: `brew install maven`

## Source Url Mapping

The app is a small demo of a java servlet app.  Here's the source code to url mapping:

Source | Url | Url2
--- | --- | ---
src/main/java/Hello.java  | localhost:8102/demo/Hello | localhost:8102/
src/main/webapp/index.jsp | localhost:8102/demo/index.jsp | localhost:8102/index.jsp

## Testing Locally with Docker

The app is also dockerized so you can test this via docker.

## Build

The build script uses `mvn package` to produce a demo.war file and then bundles it with a Docker image that runs Tomcat.  Usage:

    bin/build

## What happened

* mvn package was ran and the `target/demo.war` was moved into `pkg/demo.war`
* a docker image was built which copied the `pkg/demo.war` to `/usr/local/tomcat/webapps/demo.war`. Check out the [Dockerfile](Dockerfile) for details.

Here's an example of some things to check after running the build script:

    $ ls pkg/demo.war
    pkg/demo.war
    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    java-backend           latest              88092dfb7325        6 minutes ago       591MB
    tomcat              9.0                 a92c139758db        2 weeks ago         558MB
    $

## Run

Here are the summarized commands to run and test that Tomcat is serving the war file:

    bin/build # build Docker image
    docker run --rm -p 8102:8102 -d --name java-backend java-backend:latest
    docker exec -ti $(docker ps -ql) bash
    curl localhost:8102/
    curl localhost:8102/demo/Hello
    curl localhost:8102/demo/index.jsp
    exit
    docker stop java-backend

Then you can hit the the [HOSTNAME]:8102/demo/Hello and to verify that Tomcat is servering the demo.war file.  You should see an html page that says "Hello World".  The output should look similar:

    $ bin/build # build Docker image
    $ docker run --rm -p 8102:8102 -d --name java-backend java-backend:latest
    2ba7323481fa5c4068b90f2edf38555d9551303e9c2e4c27137ab0545688555b
    $ docker exec -ti $(docker ps -ql) bash
    root@2ba7323481fa:/usr/local/tomcat# curl localhost:8102/
    message from java backend: src/main/java/Hello.java
    root@2ba7323481fa:/usr/local/tomcat# curl localhost:8102/demo/Hello
    message from java backend: src/main/java/Hello.java
    root@2ba7323481fa:/usr/local/tomcat# curl localhost:8102/demo/index.jsp
    <html>
    <body>
    <h2>Hello World index.jsp!</h2>
    </body>
    </html>
    root@2ba7323481fa:/usr/local/tomcat# exit
    exit
    $ docker stop $(docker ps -ql)
    2ba7323481fa
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    $

## Kubes Deploy

Note: You'll need to have a docker repo that you can push to set up. The example here uses a google GCP repo.

    $ kubes deploy
    => docker build -t gcr.io/google_project/java-backend:kubes-2020-07-18T21-35-07-275a1c8 -f Dockerfile .
    Pushed gcr.io/google_project/java-backend:kubes-2020-07-18T21-35-07-275a1c8 docker image.
    Docker push took 1s.
    Compiled  .kubes/resources files to .kubes/output
    Deploying kubes resources
    => kubectl apply -f .kubes/output/shared/namespace.yaml
    namespace/java-backend created
    => kubectl apply -f .kubes/output/shared/network_policy.yaml
    networkpolicy.networking.k8s.io/web created
    => kubectl apply -f .kubes/output/web/service.yaml
    service/web created
    => kubectl apply -f .kubes/output/web/deployment.yaml
    deployment.apps/web created
    $

Note, a NetworkPolicy has been created that allows traffic from the java-backend and java-frontend namespaces only. This will be test by deploying the java-frontend app later.

## Test

List the resources:

    $ kubectl config set-context --current --namespace=java-backend # kubens java-backend
    $ kubectl get pod,svc
    NAME                       READY   STATUS    RESTARTS   AGE
    pod/web-6b58688558-nz567   1/1     Running   0          117s

    NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    service/web   ClusterIP   192.168.0.42   <none>        80/TCP    117s

Grab the pod name and exec into it to test.

    $ kubectl exec -ti pod/web-6b58688558-nz567 sh
    # curl localhost:8102
    message from java backend: src/main/java/Hello.java
    # curl 192.168.0.42
    message from java backend: src/main/java/Hello.java
    # curl web.java-backend
    message from java backend: src/main/java/Hello.java
    #

## Cleanup

Note, if you're still testing the java-frontend app, don't delete this app yet until. The frontend app talks to this backend app.

    $ kubes delete -y
    Compiled  .kubes/resources files to .kubes/output
    => kubectl delete -f .kubes/output/web/deployment.yaml
    deployment.apps "web" deleted
    => kubectl delete -f .kubes/output/web/service.yaml
    service "web" deleted
    => kubectl delete -f .kubes/output/shared/network_policy.yaml
    networkpolicy.networking.k8s.io "web" deleted
    => kubectl delete -f .kubes/output/shared/namespace.yaml
    namespace "java-backend" deleted
    $

## Initial Generation Notes

Here are some notes on the initial generation of the project. The initial files and project structure was generated with the `mvn archetype:generate` command.  Note, you do not have to run the command it is just noted here for posterity.  More info: [Creating a webapp](https://maven.apache.org/plugins-archives/maven-archetype-plugin-1.0-alpha-7/examples/webapp.html) and [Introduction to the Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html).

Change were made like adding a simple [Hello.java](src/main/java/Hello.java) Servlet class.

The original command was:

    mvn archetype:generate \
      -DinteractiveMode=false \
      -DgroupId=com.domain \
      -DartifactId=demo \
      -DarchetypeArtifactId=maven-archetype-webapp

