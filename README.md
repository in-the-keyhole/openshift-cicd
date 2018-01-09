## CI/CD architecture in Openshift

Application in Openshift will be segregated in many projects, each project is isolated from other.

![CI/CD Architecture](http://www.plantuml.com/plantuml/svg/ZLF9JiCm4BtdAwpUezvHgOfQYKL2W7hWW3Xu6HBJU94pHeeG_yxOf8i3gl31ohmtdlTcxAnwv06ZwIfqdg5ZmY4wmvGE854xM_KxRJqFt33FvOutiCMX0vReActSDXGs6jbBnSQr4Cjh0W9ujvYBvG6_f7K8QlRWmVQjVaE6O7p74VeJTkjYaC2aKv3HrxmV9PMJEmXj5ANm9iCtKPnLVhxQFfA2vU4f2c3QK6EZYknSL6pczkPgsSKU8SpOPywOs313gNy_dFJbWtkdp7DMA3-hzoLMJcJQkzHIbRBhs_bwyn-zEx1qe-MWnp7yFMRuc23qP1fjcmbAqghAM7eBYxfEaxbWiVyfUxojNnG52Siz7T4SrlKo6I1OHDsfkGBVkQ7aHpkKGnQCmydnv6l2fOfrq4sBnN7woFGUEQIC9HB_LQt0AlfVlW40 "OpenShift CI/CD Architecture")

**Openshift project**
> This is the default project that comes by default with a classic Openshift installation.
It contains images and templates supported by RedHat.

**My-Openshift project**
> The idea of this project is to mimic what is Openshift project but for our specific needs.
It will contains base image, templates, and secrets for private registry.

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

Then when we want to import an image from our private registry, we will use the reference policy `local`, so other pod will not fetch image
from it directly, instead the image will be cached in the openshift internal registry and the other pod will retrieve their image from it.

This configuration allow us to not provide docker secret in each project and so we can enforce which service account has the
right to pull an image from our private registry without sharing the secret in every project.

```sh
oc import-image ${image}:${version} --from=${image}:${version} -n my-openshift -reference-policy=local --confirm
```

Once we have imported the image, we have to allow other project to pull image from the `my-openshift` project.

To simplify this management, we can also add the ability to all service account in project `project-a` to pull image from project `my-openshift`:

```sh
oc policy add-role-to-group system:image-puller system:serviceaccounts:project-a -n my-openshift
```

### CI/CD project

We will create a cicd project to install our Jenkins, Nexus and SonarQube and whatever else is necessary for our CI/CD infrastructure.
The cicd project should be able to retrieve image from this project.

```sh
oc new-project cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:cicd -n my-openshift
```

The rest of this documentation assumes you run command in the project cicd. To ensure it, run: `oc project cidc`

#### Components installation

1. [Jenkins](https://github.com/arnaud-deprez/jenkins-docker-openshift)
1. [Nexus 3](https://github.com/arnaud-deprez/nexus3-docker)
1. [SonarQube](https://github.com/arnaud-deprez/sonarqube-docker)
