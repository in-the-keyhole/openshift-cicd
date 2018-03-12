##### 0. configure command line 
```sh
eval $(minishift oc-env)
eval $(minishift docker-env)
```

##### 1. Login as cluster administrator
```sh
oc login -u system:admin
oc project cicd
# not sure if this is needed, or if oc login does that
docker login -u system -p $(oc whoami -t) $(minishift openshift registry)
```

##### 2. Create Openshift projects 
```sh
oc new-project cicd
oc new-project dev 
oc new-project stage
```
##### 3. and grant permissions 
Note: the format for *add-role-to-user* `system:serviceaccount:PROJECT:SERVICE-ACCOUNT-NAME`

```sh
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n stage

# verification 
oc policy can-i edit system:serviceaccount:cicd:jenkins
?oc describe policyBindings :default -n dev
```


##### 4. Create Jenkins, SonarQube and PACT servers 
```sh
oc new-app jenkins-ephemeral (TODO: update to use custom image)
oc describe sa jenkins


#SonarQube (WIP)
oc new-build https://github.com/in-the-keyhole/openshift-sonarqube-s2i.git \
     --strategy=docker \
     --to=sonarqube-ocp:6.7-alpine \
     --name=sonarqube-ocp

#follow the build
oc logs -f bc/sonarqube-ocp

oc new-app -f https://raw.githubusercontent.com/in-the-keyhole/openshift-sonarqube-s2i/master/openshift/sonarqube-postgresql-template.yml -p SONARQUBE_VERSION=6.7 -p POSTGRESQL_PASSWORD=sonar
     

#Postgres (for Pact Server DB)
oc new-app registry.access.redhat.com/rhscl/postgresql-96-rhel7 -e POSTGRESQL_USER=pactuser -e POSTGRESQL_PASSWORD=pactpass -e POSTGRESQL_DATABASE=pactdb -n cicd

#verify Postgresql
oc status 
oc describe po postgresql-96-rhel7
oc describe svc postgresql-96-rhel7

#Pact Broker
oc create -f https://raw.githubusercontent.com/in-the-keyhole/openshift-cicd/master/templates/pact-server-template.yaml?token=ABVra3oqKy2LfpP9VpomBultugGpPZdGks5artk0wA%3D%3D -n cicd

#verify Pact Broker
oc status 
oc get po  | grep pact-broker
(select the pod without "build" in the name. In one case, it was pact-broker-1-k2z5d
oc logs pact-broker-1-k2z5d -c pact-broker
oc describe po pact-broker-1-k2z5d
```



