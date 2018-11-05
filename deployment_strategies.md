
# Deployment Strategies (CLI)
OpenShift supports multiple deployment and build strategies. This demo covers only few of them.

## Setup the environment
Create a new project for this application:

```
oc new-project demo-build --display-name='Test Build Strategies'
```

## Deploy an App from Source
The  default simplest way to deploy an application from source is to point directly to the soruce repository

```
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1 
```

When executing this, OpenShift check the source code and identifies the proper _s2i_ strategy to use. In this particular case there is a _Dockerfile_ with the code so it will use the _s2i_ docker build strategy. Meaning, it will build the application from source but using _docker build_

The output will be similar to this:

```
$ oc new-app https://github.com/williamcaban/podcool.git --name=myapp1
--> Found Docker image 408808f (11 days old) from Docker Hub for "python:3-alpine"

    * An image stream will be created as "python:3-alpine" that will track the source image
    * A Docker build using source code from https://github.com/williamcaban/podcool.git will be created
      * The resulting image will be pushed to image stream "myapp1:latest"
      * Every time "python:3-alpine" changes a new build will be triggered
    * This image will be deployed in deployment config "myapp1"
    * Port 8080/tcp will be load balanced by service "myapp1"
      * Other containers can access this service through the hostname "myapp1"
    * WARNING: Image "python:3-alpine" runs as the 'root' user which may not be permitted by your cluster administrator
```

From the previous output we can see it is using the Docker build strategy. OpenShift analyze the Dockerfile and display any warning. In this example, the _Dockerfile_  is pointing to a Python upstream Docker Hub base image that is running as root. The warning message seen here is because, by defuault, OpenShift does not run container images running as run. This is something the cluster-admin may choose to disable for a user, group or at cluster level.

To track the build progress use the following command:

```
oc logs -f bc/myapp1
```

A sample output will be similar to this
```
...
 ---> Running in 39f16d4ae437
 ---> 60508bc80e05
Removing intermediate container 39f16d4ae437
Successfully built 60508bc80e05
Pushing image 172.30.1.1:5000/demo-build/myapp1:latest ...
Pushed 0/12 layers, 1% complete
Pushed 1/12 layers, 25% complete
Pushed 2/12 layers, 25% complete
Pushed 3/12 layers, 25% complete
Pushed 4/12 layers, 34% complete
Pushed 5/12 layers, 42% complete
Pushed 6/12 layers, 50% complete
Pushed 7/12 layers, 63% complete
Pushed 8/12 layers, 73% complete
Pushed 9/12 layers, 80% complete
Pushed 10/12 layers, 88% complete
Pushed 11/12 layers, 96% complete
Pushed 12/12 layers, 100% complete
Push successful

```

## Forcing Deployment Strategy
By default, the ``oc new-app`` support strategies: docker, pipeline and source

The same demo app forcing to use the docker strategy. (This is the same one auto selected in the previous example):
```
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1-docker --strategy=docker
```

The same demo app forcing to use the source strategy:
```
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1-source --strategy=source
```

In the previous example, when using the *``--strategy=source``*, since no language type was specified, OpenShift will determine the language by inspecting the code repository. Because the code repository contains a ``requirements.txt``, it will subsequently be interpreted as including a Python application. When such automatic detection happens, ``python:latest`` will be used as the default image.

## Deploy from existing local image
Containers build by the system are stored in the local repository. To list the existing images in the project we have to query the _Image Stream_

```
oc new-app -S --image-stream=<name>
```

An example output is the following:

```
$ oc new-app -S --image-stream=myapp
Image streams (oc new-app --image-stream=<image-stream> [--code=<source>])
-----
myapp1
  Project: demo-build
  Tags:    latest
myapp1-docker
  Project: demo-build
  Tags:    latest
myapp1-py27
  Project: demo-build
  Tags:    latest
myapp1-source
  Project: demo-build
  Tags:    latest
myapp1-v2
  Project: demo-build
  Tags:    latest 
```
This query returned all the _Image Streams_ that started with the name _myapp_.

To deploy an app using an existing image stream:
```
oc new-app --image-stream=myapp1 --name=myapp-from-stream
```

## Deploy from external Docker registry
OpenShift can deploy existing Docker images directly from repositories.
```
oc new-app --docker-image=quay.io/redhat/podcool --name=myapp-quay
```

## Choosing a Specific Version of the Base Container
Should there be the need to select a specific Python version, lets say python 2.7, when using ``oc new-app``, you should instead use the syntax:

```
oc new-app python:2.7~https://github.com/williamcaban/openshift-container-name-demo.git --name=myapp1-py27 --strategy=source
```

## Passing Environment Variables
Should there is the need to pass environment variables during the deployment, any key/value pair appended to the deployment command will be interpreted as environment variables and will pass those the resulting container.

Using the same demo application, we can force the application to run in a different version.
```
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1-v2 --strategy=source APP_VERSION=v2
```
## Additional References

More information about [advanced deployment strategies]( https://docs.openshift.com/container-platform/3.11/dev_guide/deployments/advanced_deployment_strategies.html) is available on the official documentation.

## Cleaning the Environment
To remove the project with all the deployments used int his lab execute the following command

```
oc delete project demo-build
```