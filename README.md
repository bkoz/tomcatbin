# tomcatbin#
#
# Binary deployment from a local git repo to a Tomcat container using OpenShift v3.3
#

#
git clone https://github.com/bkoz/tomcatbin.git

oc new-project binary

oc new-build --image-stream=jboss-webserver30-tomcat7-openshift --name=blue --binary=true

oc start-build blue --from-repo=tomcatbin

oc get is

oc create deploymentconfig blue --image=<registry-service-ip>:5000/binary/blue:latest

oc expose dc blue --port=8080

oc expose svc blue --path=/sample

#
# Before defining the production route, delete the blue or blue route 
# created above to avoid router confilcts.
#
oc expose svc blue --name=production --hostname=production.ose-apps.haveopen.com --path=/sample

#
# After each new build finished, a manual deployment is necessary unless an auto trigger is setup.
oc deploy blue --latest

#
# Optionally create an auto trigger in the deployment config. 
oc edit dc/blue

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
