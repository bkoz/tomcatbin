# Binary deployment from a local git repo to a Tomcat container using OpenShift v3.3

Edit the SUBDOMAIN variable to match your OpenShift DNS wildcard subdomain.

`SUBDOMAIN=ose-apps.haveopen.com`

`git clone https://github.com/bkoz/tomcatbin.git`

`oc login`

`oc new-project binary`

`oc new-build --image-stream=jboss-webserver30-tomcat7-openshift --name=blue --binary=true`

`oc start-build blue --from-repo=tomcatbin`

`oc get is`

`oc create deploymentconfig blue --image=<registry-service-ip>:5000/binary/blue:latest`

`oc expose dc blue --port=8080`

`oc expose svc blue --name=blue --hostname=blue.$SUBDOMAIN --path=/sample`

Get the hostname of the route and visit your application using a web browser.

`oc get route`

After each new build finished, a manual deployment is necessary unless an auto trigger is setup.

`oc deploy blue --latest`


### Optional steps: 


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

#### Blue/Green Deployment

To simulate a blue/green deployment, copying the blue or green .war file
into the deployments directory as `ROOT.war`

`cp blue.war deployments/ROOT.war`

`oc start-build --from-repo=tomcatbin`

`oc deploy blue --latest`

`oc delete route blue`

`oc expose service blue --hostname=production.$SUB_DOMAIN`


Before defining the production route, delete the blue or green test route 
created above to avoid router confilcts.

`oc delete route blue`

`oc expose svc blue --name=production --hostname=production.$SUBDOMAIN `
