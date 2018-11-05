# Testing Pods Resiliency
In order to see how the applications are self-healed in OpenShift lets start by doing a continue request to the demo app and observe its behavior as we execute additional steps. 

## Initial setup
Setup a project for this test, deploy the ``podcool`` app and create the external route for it.
```
# Create new project
oc new-project pod-healing --display-name='Demo Pod Self Healing'

# Deploy new podcool app
oc new-app https://github.com/williamcaban/podcool.git --name=myapp1 --strategy=source

# Create external route
oc expose svc/myapp1 --name=myroute
```

## Monitor the availability of the route
***In a different terminal*** run the following loop to get the text output displaying the name of the pod and the version of the app handling the request.  Use the _/hello_ path to get the desired output.

Run the following loop ***in another terminal***:
```
while sleep 1; do curl -w "\n" http://$(oc get route myroute --template='{{ .spec.host }}'/hello); done
```

The output should display a _Hello_ message with the _name_ of the pod and _version_ of the application similar to this:

```
$ oc get pods
NAME             READY     STATUS      RESTARTS   AGE
myapp1-1-build   0/1       Completed   0          40m
myapp1-1-gx9vs   1/1       Running     0          39m

$ while sleep 1; do curl -w"\n" http://$(oc get route myroute --template='{{ .spec.host }}'/hello); done
Hello from myapp1-1-gx9vs v1
Hello from myapp1-1-gx9vs v1
Hello from myapp1-1-gx9vs v1
Hello from myapp1-1-gx9vs v1
Hello from myapp1-1-gx9vs v1
```

Leave this loop running the terminal and return to the original terminal with the OpenShift client to continue the lab.

## Scaling the Application

Scale to 3 replicas and validate pods have been created
```
# Validate the numbers of pods running your app
oc get pods -l app=myapp1

# Scale the number of pods
oc scale --replicas=3 dc/myapp1

# Validate the new pods are in Running status
oc get pods -l app=myapp1
```

The output should look similar to this:
```
$ oc get pods -l app=myapp1
NAME             READY     STATUS    RESTARTS   AGE
myapp1-1-lttx5   1/1       Running   0          <invalid>
myapp1-1-m55gc   1/1       Running   0          <invalid>
myapp1-1-sq6kf   1/1       Running   0          <invalid>
```

The output in terminal monitoring the route availability should see the request being answered by a different pod each time (a simple round-robin distribution)

```
...
Hello from myapp1-1-sq6kf v1
Hello from myapp1-1-lttx5 v1
Hello from myapp1-1-m55gc v1
Hello from myapp1-1-sq6kf v1
Hello from myapp1-1-lttx5 v1
Hello from myapp1-1-m55gc v1
...
```

## Testing Self-Healing

Identify and destroy one of the Pods and monitor how the system remediate.
```
oc get pods -l app=myapp1

oc delete po/<name-of-pod>

oc get pods -l app=myapp1
```

The results should be similar to the following output:

```
# List the pods serving your application
$ oc get pods -l app=myapp1
NAME             READY     STATUS    RESTARTS   AGE
myapp1-1-lttx5   1/1       Running   0          6m
myapp1-1-m55gc   1/1       Running   0          6m
myapp1-1-sq6kf   1/1       Running   0          6m

# Delete one of the pods
$ oc delete po/myapp1-1-lttx5
pod "myapp1-1-lttx5" deleted

# Monitor how the system remediate the number of active pods
$ oc get pods -l app=myapp1
NAME             READY     STATUS        RESTARTS   AGE
myapp1-1-6k28r   1/1       Running       0          <invalid>
myapp1-1-lttx5   0/1       Terminating   0          6m
myapp1-1-m55gc   1/1       Running       0          6m
myapp1-1-sq6kf   1/1       Running       0          6m
$
```

The curl loop output should show the new pod serving requests:

```
...
Hello from myapp1-1-sq6kf v1
Hello from myapp1-1-lttx5 v1
Hello from myapp1-1-m55gc v1
Hello from myapp1-1-sq6kf v1
Hello from myapp1-1-lttx5 v1
Hello from myapp1-1-m55gc v1
...
```

## Cleaning the Environment
To remove this project and the corresponding objects 
```
oc delete project pod-healing
```
To stop the curl loop in the terminal monitoring the route press _CTRL-C_