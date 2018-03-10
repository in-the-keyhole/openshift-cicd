#### Create Docker image for Pact-JVM

The Docker image provided by the Pact project runs as root, which is not allowed for Openshift Docker images

To create an OpenShift-compatible Pact Docker image, use source-to-image (s2i)

```sh
s2i [ask Jaime for details]
```

then push to Keyhole Dockerhub

```sh

```