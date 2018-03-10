## Reference OpenShift CI/CD architecture

![CI/CD Architecture](http://www.plantuml.com/plantuml/png/06re408pKM2qus_nYtVSjwStlZcHIF70FG5pv_t-gvWGmH75-wsPqJ1p7YxNItzw0TG0LgnKhKXVpsk4tEBN78q2h30CguWJY7mB0WPZBXfLo9tFL0921vb-ro0uOjbj08000WAs-VxW5CavYViMxVdV0Ws5Q5Z5hsA2z2o4YDu2k_6QK5eVOJypOy3bpIftmyVggUYRFeaI-uSsHxqOvLk15Ztf_dluV_ED8oxwbpZ1xYPf9TyGaPi5yMW6v1Zs5w0hOm4T7qyQ1IQQq-TWGlzL_MR55uE9y2_P7C8loyqJOKlmKjWgLJUz9Nc_2bWGy1zNY1BzYk8R9N_IwbDYAcd3_45Jyef9hYgkdx-I5PhkH_ORuO6lISc6x1RPJvftNA7EU8KSZhxTX-vnOvAxWtIW0FI9zdsYunJhuS37NshNKyFwntCyBpKPcJ3XW3RtL1s6sX3OYI0mT7ml57Ky1njZ8TZtIaIEp30C0)

**Git Repo**
> The Git repo contains the application source code, along with the templates used to instantiate various OpenShift Build configurations  (https://github.com/in-the-keyhole/openshift-cicd)

**DockerHub**
> DockerHub contains regularly updated versions of Jenkins and SonarQube, each containing the latest plugins for Static Application Security Testing (SAST), mutation testing (pitest) and third-party-dependency monitoring (dependency-check).  
1. keyholeDockerUrl/jenkins-cicd
1. https://hub.docker.com/r/owasp/sonarqube/

**Openshift project**
> This is the default project that comes with Openshift Origin. During setup, we will create ImageStreams for Custom Jenkins, Custom SonarQube, and PACT-jvm. We will also grant permissions necessary to promote validated images through the dev/staging/prod environments (which are OpenShift projects)

**my-openshift project**
> This project contains BuildConfigurations for client application source code. A large number of source projects can be created in this project.  During setup, we will The idea of this project is to mimic what is Openshift project but for our specific needs. It will contains base image, templates, and secrets for private registry.

**Dev project**
> The Dev project is where a validated build image will be tagged and pushed. The SonarQube Quality Gate will be the arbiter of whether a build is "validated". If the Quality Gate passes, the image is validated. The linkage necessary for Jenkins to know if the SonarQube analysis validated the build is pre-configured within the custom images. (TODO: not fully pre-configured yet)

> Consulting Note regarding Shared Service images in the Dev Project: When Shared Services are being built, care must be given to balance the needs of Shared Service Consumers (needing fewer, more stable versions) and Shared Service Providers under development (needing to deploy updates to test with *their* collaborators).  Best practice is to implement Consumer Driven Contracts (CDC) testing within the Build pipeline, using a tool such as PACT. With CDC tests in place, the need for lengthy, manual integration tests are reduced.


**Staging project**
> This is where Business Facing Tests, Live Contract Tests, and other End-To-End tests are run. These tests are usually a combination of automated and manual tests. If tests passed, the image can be tagged. If all tests are automated, the possibility exists that the image could be tagged as part of the build pipeline and deployed into production automatically. Until the client develops high confidence in the automated test suite, there will be human involved in the deployment step. The human involvement can be as minimal as simply logging into Jenkins and starting a "deploy" job. (TODO: the deploy job has not yet been configured into the Custom Jenkins image)



## Setup
### Cluster Admin 
When first on-site, the cluster administrator must prepare the environment by doing the following:
- create the cicd, dev and stage Openshift projects 
- import the custom Docker images into the Openshift image registry
- 

[script](cluster-administration.md)
[script](cluster-cleanup.md)

### CI/CD project

We will create a cicd project to install our Jenkins and SonarQube servers, possibly including Nexus if needed.
The cicd project should be able to retrieve images from this project.

```sh
oc new-project cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:cicd -n my-openshift
```

The rest of this documentation assumes you run commands in the project cicd. To ensure it, run: `oc project cidc`

### My project
TODO: need to validate this section

Similar to the Openshift project, we will create a my-openshift project to store our custom templates and eventually our images from our private registry.

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




#### Docker Image maintenance

1. [Create Custom Jenkins Docker image](https://github.com/in-the-keyhole/openshift-jenkins-s2i)
1. [SonarQube](https://github.com/OWASP/sonarqube)
1. [Build PACT Docker image](https://github.com/in-the-keyhole/openshift-cicd/images/pact-s2i.md)

if necessary, also install [Nexus 3](https://github.com/arnaud-deprez/nexus3-docker)

To edit the diagram, use [PlantUML](http://www.plantuml.com/plantuml/uml/bLFBJiCm4BpdAopkKU-eL4MjnEC2A0SE2277NbBJ-25dZLGX_fsrIHESLWLyiB8pQy_iUhFia7iCkYtGEeQMrHRHQYQL1u7AcgBRAkEuvvevBhQyWGftBR185t7Zfg6mKSYU2jQlURsu8i23i_DPlHZm2rf3KB8x1wRRg5Ta2Dgr7A7xmLOsU05CM0a9VKxxA2cjs8BVX3eNNYVuEjGdbHylgtjICDm_X1gOqbHOho9Q6oGxitjpfMX3X-3Fs4VAcDWOyd8ROstEspVA_gqHLKfEHhgZLDwZQJVqBjVNknVx7mjq_a2RiDEYGWcPowvkPglrN_HkmT1WvU_TlyRnruRt2KAJsoZJJ52IbaN2uaZEorJ5EGlTzxNZq1nG54GirJIYOnEFa2aioqLqHt0TL2jd4bnhTpdVUwKSygSjJev7qnCktbXwrMmmEquVYTuZGqf_HUG_YoKOIV_q5m00)
