# Binary deployments to Tomcat with OpenShift v3.5

## How to deploy a war file from a local directory to a Tomcat container.
Edit the `SUBDOMAIN` and `OPENSHIFT_SERVER` variables to match your environment.
```
SUBDOMAIN=ose-apps.haveopen.com
OPENSHIFT_SERVER=masteroselab-bkocp33-utzuqe0l.srv.ravcloud.com
```

Clone this git repo locally.
```
git clone https://github.com/bkoz/tomcatbin.git
cd tomcatbin
```
Login to OpenShift, create a project and  new build then start the build.
```
oc login $OPENSHIFT_SERVER:8443

oc new-project binary
oc new-build --image-stream=jboss-webserver30-tomcat8-openshift --name=sample --binary=true
oc start-build sample --from-dir=source
```

Follow the build logs and wait for the build and registry push to suceed.
```
oc logs bc/sample --follow
```
```
...
Pushed 7/7 layers, 100% complete
Push successful
```

Create a deployment config. 
```
image=`oc get is sample --template={{.status.dockerImageRepository}}`
oc create deploymentconfig sample --image=$image:latest
```

Follow the deployment logs and wait for the deployment pod to suceed.
```
oc logs dc/sample --follow
```
```
...
--> Waiting up to 10m0s for pods in deployment sample-1 to become ready
--> Success
```

Once the sample pod is running, follow the logs and wait for Catalina Server to finish starting.
```
2016-12-03 08:03:57,950 [main] INFO  org.apache.catalina.startup.Catalina- Server startup in 5913 ms
```

Expose the sample application and create a route for it.
```
oc expose dc sample --port=8080
oc expose svc sample --name=sample --hostname=sample.$SUBDOMAIN 
```

Get the hostname of the route and visit your application using a web browser.
```
oc get route
```
```
NAME      HOST/PORT                      PATH      SERVICES   PORT      TERMINATION
sample    sample.ose-apps.haveopen.com             sample     8080      
```

### Optional steps: 

#### Re-builds
After each subsequent start-build finishes, a manual deployment is necessary unless an auto trigger is setup.
```
oc rollout latest dc/sample
```

#### Triggers
Create an auto trigger in the deployment config so the application is automatically 
deployed after a new build. 

`oc edit dc/sample`

```
triggers:
- imageChangeParams:
    automatic: true
    containerNames:
    - default-container
    from:
      kind: ImageStreamTag
      name: sample:latest
      namespace: binary
  type: ImageChange
- type: ConfigChange
```

