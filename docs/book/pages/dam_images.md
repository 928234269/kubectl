{% panel style="danger", title="Proposal Only" %}
Many of the features and workflows.  The features that must be implemented
are tracked [here](https://github.com/kubernetes/kubectl/projects/7)
{% endpanel %}

{% panel style="info", title="TL;DR" %}
- Set the Tag for all Container Images matching a Name with `images`
- Set the Tag for Container Images from another source (e.g. Git Hash)
- Set the name for Container Images
{% endpanel %}

# Container Images

## Motivation

It may be useful to define the tags that are used for specific container 
images used across many Workloads.

- The same image tag is used for multiple different container images 
- The same container image is used in multiple containers or Workloads
- Increase visibility of the versions of container images being used within
  the project
- Setting the image tag from external sources - such as environment variables
- Copy or Fork an existing Project and change the Image Tag for a container

It may also be useful to replace the name of an image.

- Replace the image used by a generic base.
- Change the container registry

See [Bases and Variations](project_variants.md) for more details on Copying Projects.

## images

It is possible to set images or image tags for container images by through 
`kustomization.yaml` using the `images` field.  When `images` are 
specified, Apply will update all images matching the name
that match the name.

{% method %}

**Example:** Set the `images` specified in the `kustomization.yaml` on the 
container images specified in `deployment.yaml`

Apply will set the `nginx` image to have the tag `1.8.0` - e.g. `nginx:1.8.0`.
This will set the tag for *all* untagged images matching the *name*.

{% sample lang="yaml" %}
**Input:** The kustomization.yaml and deployment.yaml files

```yaml
# kustomization.yaml
apiVersion: v1beta1
kind: Kustomization
images:
  - name: nginx
    newTag: 1.8.0
resources:
- deployment.yaml

# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Applied:** The Resource that is Applied to the cluster

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      # The image has been changed to include the tag
      - image: nginx:1.8.0
        name: nginx
```
{% endmethod %}


## Setting a Tag

{% method %}
The tag for an image may be set by specifying `newTag` and the name of the container image.
{% sample lang="yaml" %}
```yaml
# kustomization.yaml
apiVersion: v1beta1
kind: Kustomization
images:
  - name: mycontainerregistry/myimage
    newTag: v1
```
{% endmethod %}

## Setting a Digest

{% method %}
The digest for an image may be set by specifying `digest` and the name of the container image.
{% sample lang="yaml" %}
```yaml
# kustomization.yaml
apiVersion: v1beta1
kind: Kustomization
images:
  - name: alpine
    digest: sha256:24a0c4b4a4c0eb97a1aabb8e29f18e917d05abfe1b7a7c07857230879ce7d3d3
```
{% endmethod %}

## Setting a new image name

{% method %}
A new image name can be set by specifying `newName`.
{% sample lang="yaml" %}
```yaml
# kustomization.yaml
apiVersion: v1beta1
kind: Kustomization
images:
  - name: alpine
    newName: gcr.io/my-project/alpine
    newTag: v1
```
{% endmethod %}

## Setting a Tag from the latest commit SHA

{% method %}
A common CICD pattern is to tag container images with the git commit SHA of source code.  e.g. if
the image name is `foo` and an image was built for the source code at commit `1bb359ccce344ca5d263cd257958ea035c978fd3`
then the conatiner image would be `foo:1bb359ccce344ca5d263cd257958ea035c978fd3`.

A simple way to push an image that was just build without manually updating the image tags is to use the
`kustomize edit set imagetag` command to update the tags for you.

**Example:** Set the latest git commit SHA as the image tag for `foo` images.

{% sample lang="yaml" %}
```bash
$ kustomize edit set imagetag foo:$(git log -n 1 --pretty=format:"%H")
$ kubectl apply -f .
```
{% endmethod %}

## Setting a Tag from an Environment Variable

{% method %}
It is also possible to set a Tag from an environment variable using the same technique for setting from a commit SHA.

**Example:** Set the tag for the `foo` image to the value in the environment variable `FOO_IMAGE_TAG`.

{% sample lang="yaml" %}
```bash
$ kustomize edit set imagetag foo:$FOO_IMAGE_TAG
$ kubectl apply -f .
```
{% endmethod %}

{% panel style="info", title="Committing Image Tag Updates" %}
The `kustomization.yaml` changes *may* be committed back to git so that they
can be audited.  When committing the image tag updates that have already 
been pushed by a CICD system, be careful not to trigger new builds + 
deployments for these changes.
{% endpanel %}
