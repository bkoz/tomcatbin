# Binary deployment from a local git repo to a Tomcat container using OpenShift v3.3

Edit the SUBDOMAIN variable to match your OpenShift DNS wildcard subdomain.

`SUBDOMAIN=ose-apps.haveopen.com`

`git clone https://github.com/bkoz/tomcatbin.git`

`oc login`

`oc new-project binary`

`oc new-build --image-stream=jboss-webserver30-tomcat7-openshift --name=blue --binary=true`

`oc start-build blue --from-repo=tomcatbin`

Wait for the build to and registry push to finish.

`oc logs bc/blue`

```
...
Pushed 7/7 layers, 100% complete
Push successful
```
Create a deployment config using the image stream info.

`oc get is`

`oc create deploymentconfig blue --image=<registry-service-ip>:5000/binary/blue:latest`

Expose the blue application and create a route for it.

`oc expose dc blue --port=8080`

`oc expose svc blue --name=blue --hostname=blue.$SUBDOMAIN --path=/blue`

Get the hostname of the route and visit your application using a web browser.

`oc get route`

After each subsequent start-build finishes, a manual deployment is necessary unless an auto trigger is setup.

`oc deploy blue --latest`


### Optional steps: 

#### Blue/Green Deployment

To simulate a blue/green deployment, copy the blue or green .war file
to `ROOT.war`

`cp tomcatbin/deployments/blue.war tomcatbin/deployments/ROOT.war`

`oc start-build --from-repo=tomcatbin`

`oc deploy blue --latest`

`oc delete route blue`

`oc expose service blue --hostname=production.$SUBDOMAIN`

#### Triggers

Create an auto trigger in the deployment config so the application is automatically 
deployed after a new build. 

`oc edit dc/blue`

```
triggers:
- imageChangeParams:
    automatic: true
    containerNames:
    - default-container
    from:
      kind: ImageStreamTag
      name: blue:latest
      namespace: binary
  type: ImageChange
- type: ConfigChange
```

