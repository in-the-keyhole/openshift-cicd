##### 1. Login as cluster administrator
```sh
oc login -u system:admin
```

##### 2. Build PACT-server docker image
```sh
s2i ... (TODO: update with s2i command to build PACT. Jaime did it)
```

##### 3. Create Jenkins, SonarQube and PACT servers 
```sh
oc new-app jenkins-ephemeral (TODO: use custom image)
oc describe sa jenkins

oc new-app -f https://github.com/in-the-keyhole/openshift-cicd/templates/sonarqube-template.yaml --param=SONARQUBE_VERSION=6.7 --param=SONAR_MAX_MEMORY=1Gi
oc describe sa sonarqube

oc new-app -f https://github.com/in-the-keyhole/openshift-cicd/templates/pact-server-template.yaml
oc describe sa sonarqube
```

##### 4. Create Openshift projects 
```sh
oc new-project cicd
oc new-project dev 
oc new-project stage
```

##### 5. and grant permissions 
Note: the format for *add-role-to-user* `system:serviceaccount:PROJECT:SERVICE-ACCOUNT-NAME`

```sh
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n stage

# verification 
oc policy can-i edit system:serviceaccount:cicd:jenkins
oc describe policyBindings :default -n dev
```