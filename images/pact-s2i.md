#### Create Docker image for Pact-JVM

The Docker image provided by the Pact project runs as root, which is not allowed for Openshift Docker images

To create an OpenShift-compatible Pact Docker image, use source-to-image (s2i)
```sh
s2i build https://github.com/openshift/ruby-hello-world centos/ruby-22-centos7 pact-broker
```
Then start Postgres
```sh
docker pull registry.access.redhat.com/openshift3/postgresql-92-rhel7
```


To build and install into Openshift in one command:

```sh
oc new-app -f https://raw.githubusercontent.com/in-the-keyhole/openshift-cicd/master/templates/pact-server-template.yaml?token=ABVrazQ1_QEBm7OW_GSdAgQO3FWVY6_Uks5arVY-wA%3D%3D
```

Reference
Postgres container: https://github.com/sclorg/postgresql-container

=== to clean
oc delete all -l app=postgresql-92-rhel7