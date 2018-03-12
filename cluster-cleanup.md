##### 1. Login as cluster administrator
```sh
oc login -u system:admin
```

##### 2. Delete Jenkins, SonarQube and PACT servers 

```sh

#SonarQube
oc delete all -l app=sonarqube

#Postgresql
oc delete all -l app=postgresql-96-rhel7

#PACT broker
oc delete all -l app=pact-broker
```
