## Reference OpenShift CI/CD architecture

![CI/CD Architecture](http://www.plantuml.com/plantuml/svg/bLFBJiCm4BpdAopkKU-eL4MjnEC2A0SE2277NbBJ-25dZLGX_fsrIHESLWLyiB8pQy_iUhFia7iCkYtGEeQMrHRHQYQL1u7AcgBRAkEuvvevBhQyWGftBR185t7Zfg6mKSYU2jQlURsu8i23i_DPlHZm2rf3KB8x1wRRg5Ta2Dgr7A7xmLOsU05CM0a9VKxxA2cjs8BVX3eNNYVuEjGdbHylgtjICDm_X1gOqbHOho9Q6oGxitjpfMX3X-3Fs4VAcDWOyd8ROstEspVA_gqHLKfEHhgZLDwZQJVqBjVNknVx7mjq_a2RiDEYGWcPowvkPglrN_HkmT1WvU_TlyRnruRt2KAJsoZJJ52IbaN2uaZEorJ5EGlTzxNZq1nG54GirJIYOnEFa2aioqLqHt0TL2jd4bnhTpdVUwKSygSjJev7qnCktbXwrMmmEquVYTuZGqf_HUG_YoKOIV_q5m00 "OpenShift CI/CD Architecture")

**Git Repo**
> The Git repo will contain the Keyhole source code, along with the OpenShift templates used to create the Jenkins image and the other OpenShift artifacts (https://github.com/in-the-keyhole/openshift-cicd)

**DockerHub**
> DockerHub contains a regularly updated version of SonarQube containing the latest plugins for SAST scanning (find-sec-bugs), mutation testing (sonar-pitest) and third-party-dependency monitoring (dependency-check). https://hub.docker.com/r/owasp/sonarqube/

**my-openshift project**
> The idea of this project is to mimic what is Openshift project but for our specific needs. It will contains base image, templates, and secrets for private registry.

**Dev project**
> Dev project is where the released image for each cell will be pushed and tagged from each cell project
when developer feel confident with.
This project will also be used by developers to make some tests in an integrated environment.

**Staging project**
> This is where business e2e tests are run. If tests passed, the cell can be promoted to OPT.

**Prod project**
> This project will be used for user acceptance before going to OPS.

## Setup

### My project

Similarly to the Openshift project, we will create a my-openshift project to store our custom templates and eventually our images from our private registry.

```sh
oc new-project my-openshift
oc secrets new-dockercfg my-registry --docker-server="<url>" --docker-username="<username>" --docker-password="<password>" --docker-email="<email>"
oc secrets link default my-registry --for=pull
# to build an image
oc secrets link builder my-registry
```

Then when we want to import an image from our private registry, we will use the reference policy `local`, so other pods will not fetch image from it directly. Instead the image will be cached in the openshift internal registry and the other pod will retrieve their image from it.

This configuration allow us to not provide docker secrets in each project and so we can enforce which service account has the right to pull an image from our private registry without sharing the secret in every project.

```sh
oc import-image ${image}:${version} --from=${image}:${version} -n my-openshift -reference-policy=local --confirm
```

Once we have imported the image, we have to allow other project to pull image from the `my-openshift` project.

To simplify this management, we can also add the ability to all service accounts in project `project-a` to pull image from project `my-openshift`:

```sh
oc policy add-role-to-group system:image-puller system:serviceaccounts:project-a -n my-openshift
```

### CI/CD project

We will create a cicd project to install our Jenkins, SonarQube and whatever else is necessary for our CI/CD infrastructure.
The cicd project should be able to retrieve image from this project.

```sh
oc new-project cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:cicd -n my-openshift
```

The rest of this documentation assumes you run command in the project cicd. To ensure it, run: `oc project cidc`

#### Components installation

1. [Jenkins](https://github.com/in-the-keyhole/openshift-jenkins-s2i)
1. 
1. [SonarQube](https://github.com/arnaud-deprez/sonarqube-docker)

Note: Arnaud Deprez also included Nexus 3. I had not planned on doing that 
[Nexus 3](https://github.com/arnaud-deprez/nexus3-docker)

HT: https://github.com/arnaud-deprez/cicd-openshift/

To edit the diagram, use [PlantUML](http://www.plantuml.com/plantuml/uml/bLFBJiCm4BpdAopkKU-eL4MjnEC2A0SE2277NbBJ-25dZLGX_fsrIHESLWLyiB8pQy_iUhFia7iCkYtGEeQMrHRHQYQL1u7AcgBRAkEuvvevBhQyWGftBR185t7Zfg6mKSYU2jQlURsu8i23i_DPlHZm2rf3KB8x1wRRg5Ta2Dgr7A7xmLOsU05CM0a9VKxxA2cjs8BVX3eNNYVuEjGdbHylgtjICDm_X1gOqbHOho9Q6oGxitjpfMX3X-3Fs4VAcDWOyd8ROstEspVA_gqHLKfEHhgZLDwZQJVqBjVNknVx7mjq_a2RiDEYGWcPowvkPglrN_HkmT1WvU_TlyRnruRt2KAJsoZJJ52IbaN2uaZEorJ5EGlTzxNZq1nG54GirJIYOnEFa2aioqLqHt0TL2jd4bnhTpdVUwKSygSjJev7qnCktbXwrMmmEquVYTuZGqf_HUG_YoKOIV_q5m00)
