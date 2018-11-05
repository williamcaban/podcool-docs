# Deploying App using the ``oc`` CLI tool
Deploying an application using the ``oc`` command line tool instead of the OpenShift web console.

Make sure you are logged into the environment as _developer_:

```
oc login -u developer
```

Create a new project for this application:

```
oc new-project demo-app --display-name='My Demo App'
```

To validate the project was created list the projects and the new ``demo-app`` project should be listed there.
```
$ oc get projects
NAME         DISPLAY NAME   STATUS
demo-app     My Demo App    Active
demo-s2i     Demo s2i       Active
my-project   My Project     Active
$
```

Deploy the _podcool_ applicaiton from github running the following command:

```
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1 --strategy=source
```

* NOTE: By default there is no need to specify the *strategy* flag as OpenShift will evaluate the repo and determine the best strategy to use. Since this demo repo contains a ``Dockerfile``, by default, OpenShift will detect it and use the ``docker build strategy``. By specifying the *strategy* flag we force OpenShift to use ``s2i source build strategy``.

The ``oc new-app`` command will checkout the source code and identify it as a Python app. It will use a _build_ pod to build a new container with the appropriate Python environment setup to run the application.

To monitor the progress list the pods with the ``oc`` command and the ``-w`` option to watch until the new pod is shown in _Running_ status.

```
$ oc get pods -w
NAME             READY     STATUS      RESTARTS   AGE
myapp1-1-build   0/1       Completed   0          5s
myapp1-1-gx9vs   1/1       Running     0          <invalid>
```

Once the pod with the application is running you may find the newly created base container in the internal docker registry. OpenShift use _Image Streams_ objects to abstract the references to the container image and its tags. The _Image Stream_ object does not contain the actual image but pointer to the docker image as well as some additional metadata.

To list the Image Streams use the ``oc get imagestream`` or `oc get is`
```
$ oc get is
NAME      DOCKER REPO                       TAGS      UPDATED
myapp1    172.30.1.1:5000/demo-app/myapp1   latest    18 minutes ago

# To look inside the Image Stream
$ oc describe is/myapp1
Name:			myapp1
Namespace:		demo-app
Created:		18 minutes ago
Labels:			app=myapp1
Annotations:		openshift.io/generated-by=OpenShiftNewApp
Docker Pull Spec:	172.30.1.1:5000/demo-app/myapp1
Image Lookup:		local=false
Unique Images:		1
Tags:			1

latest
  no spec tag

  * 172.30.1.1:5000/demo-app/myapp1@sha256:da37d21c905e93a79a885ef6931d6c9f5d57a3ee94b97e8205487bc206e7df01
      18 minutes ago
$
```
Additional information about [_Image Streams_](https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/builds_and_image_streams.html#image-streams) can be found in the official documentation.

## Creating an external route for the application
When deploying an application using the ``oc new-app`` command, contrary to when using the developer console, no external route is automatically created.

To create a URL route determine the name of the service and 

- Validate there are no external routes associated to your application
```
$ oc get routes
No resources found.
```

- Find the name of the service use the ``services`` or ``svc`` option with the CLI client.
```
$ oc get svc
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
myapp1    ClusterIP   172.30.251.78   <none>        8080/TCP   23m
```

- Once the name of the service is identified create an external route to expose the service outside the cluster
```
oc expose svc/myapp1 --name=myroute
```

- Find the FQDN of the route and try accesing your application

```
oc get route

# The output will be similar to this
$ oc get route
NAME      HOST/PORT                              PATH      SERVICES   PORT       TERMINATION   WILDCARD
myroute   myroute-demo-app.192.168.64.9.nip.io             myapp1     8080-tcp                 None
wcabanba-mac:~ wcabanba$
```

Try accessing the corresponding URL from your browser.

```
# In the previous output the app URL will be 
http://myroute-demo-app.192.168.64.9.nip.io
```

To test the route from CLI using the ``curl`` command try the ``/hello`` URL of the ``podcool`` demo app

```
curl -w "\n" http://myroute-demo-app.192.168.64.9.nip.io/hello
```

## Cleaning the Environment
To remove this project and the corresponding objects 
```
oc delete project demo-app
```