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

oc new-app -f https://github.com/in-the-keyhole/openshift-cicd/templates/sonarqube-template.yaml --param=SONARQUBE_VERSION=6.7 --param=SONAR_MAX_MEMORY=1Gi
oc describe sa sonarqube

#Postgres (for Pact Server DB)
oc new-app registry.access.redhat.com/rhscl/postgresql-96-rhel7 -e POSTGRESQL_USER=pguser -e POSTGRESQL_PASSWORD=pgpass -e POSTGRESQL_DATABASE=pactdb -n cicd

#Pact Server
oc create -f https://github.com/in-the-keyhole/openshift-cicd/templates/pact-server-template.yaml -n cicd
```



