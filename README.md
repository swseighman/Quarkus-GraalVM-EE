## Quarkus on GraalVM Enterprise

GraalVM is compatible with many of today's most popular microservices frameworks, including Micronaut, Quarkus and Spring Boot. In this tutorial, we'll focus on building an application using Quarkus.  We'll create a native image, a container and ultimately deploy the container to OpenShift.

### Let's get started!

Browse to [https://code.quarkus.io/](https://code.quarkus.io/) and create a new project.

![](images/quarkus-project.png)

Group:  **com.example**  
Artifact: **quarkus-graalvm**  
Build Tool: **Maven**  
Choose **RESTEasy JAX-RS** extension


Download and unzip the `quarkus-graalvm.zip` project.

Install either [GraalVM Community Edition](https://www.graalvm.org/docs/getting-started/) or [GraalVM Enterprise Edition](https://github.com/swseighman/Installing-GraalVM-Enterprise-Edition).

Confirm your installation:

```
$ java -version

```

You can create a native image if you choose:

```
$ export GRAALVM_HOME=<path to your GraalVM install>; export GRAALVM_HOME
$ ./mvnw -DskipTests -Pnative clean package
$ target/quarkus-graalvm-1.0.0-SNAPSHOT-runner
```

You can also confirm what GraalVM distribution was used to build the native image:

```
$ strings target/quarkus-graalvm-1.0.0-SNAPSHOT-runner | grep GraalVM
```

### Build the Container Image

By default, the `.dockerignore` will exclude everything but the `target` directory during the image build so we'll need to edit `.dockerignore` and add the following exceptions:

```
!src
```
And remove:

```
*
!maven-settings.xml
```

The end result `.dockerignore` should be:

```
```

Now we can build the container image:

```
$ docker build -f src/main/docker/Dockerfile.multistage -t graalvm-quarkus .
```

```
$ docker images
```

To test:

```
$ docker run -i --rm -p 8080:8080 graalvm-quarkus
```
```
__  ____  __  _____   ___  __ ____  ______
```
```
$ curl http://localhost:8080/hello-resteasy
```

### Deploy to OpenShift Using the New Container

First, let's push our new container image to a registry.  For this example. we'll use DockerHub.

```
$ docker login
$ docker tag graalvm-quarkus:latest seighman/graalvm-quarkus
$ docker push seighman/graalvm-quarkusUsing default tag: latest
```
Over at [hub.docker.com](hub.docker.com), you can verify the image was added to your repository.

![](images/ocp-9.png)

Next, you'll need to create a (free) OpenShift sandbox account. Start [here](https://developers.redhat.com/developer-sandbox).

Once your sandbox is enabled, login to your environment.  Click on `Topology` and then choose `Container Image`.

![](images/ocp-1.png)

Enter the repository name, an application name and the name.  Click on `Create`.

![](images/ocp-2.png)

![](images/ocp-3.png)

After a minute or so, your application should appear in the `Topology`.  Click on the route (in the upper right-hand of the `Topology` icon) to access your application.

![](images/ocp-4.png)

![](images/ocp-5.png)

To access the endpoint, add `/hello-resteasy` to the route.

Example: 

```
http://graalvm-quarkus-app-seighman-code.apps.sandbox.x8i5.p1.openshiftapps.com/hello-resteasy
```

![](images/ocp-6.png)

You can also scale the pods (up/down) and view other characteristics of your application by clicking on the application icon.

![](images/ocp-7.png)

![](images/ocp-8.png)

### (Alternaive) Using the Quay Registry

```
$ docker ps -l
```
```
$ docker commit 50ab0ee29ef5 quay.io/scottseighman/graalvm-quarkus
```
```
$ docker push quay.io/scottseighman/graalvm-quarkus
```