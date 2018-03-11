##### 1. Login as cluster administrator
```sh
oc login -u system:admin
```

##### 2. Delete Jenkins, SonarQube and PACT servers 

```sh

#Postgresql
oc delete all -l app=postgresql-96-rhel7
```
