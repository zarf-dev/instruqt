---
slug: create-zarf-package
id: haa2sqszamql
type: challenge
title: Create a Zarf Package
teaser: Create a zarf package containing all artifacts and dependencies for portability.
notes:
- type: text
  contents: |-
    ![zarf-logo.png](https://play.instruqt.com/assets/tracks/ik0jeigqmuwc/e9d6f48a7da2d12026ba50b58c06d271/assets/zarf-logo.png)
    # Create a Zarf Package
    Zarf packages all the contents of your application into a single OCI artifact that can then be deployed into an airgapped environment. In this tutorial, you'll learn the basics of creating your own Zarf package.
tabs:
- id: 6bruax7vvpy0
  title: Code Editor
  type: code
  hostname: kubernetes-vm
  path: /root/zarf-package/
- id: 152sjjw8hkd1
  title: Terminal
  type: terminal
  hostname: kubernetes-vm
  workdir: /root/zarf-package/
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: null
---
Getting Started with Zarf
===
![zarf.png](https://play.instruqt.com/assets/tracks/ik0jeigqmuwc/cca3978f9217d4cfdbcf6f22c81d0d43/assets/zarf.png)

Zarf is an open source software packaging tool that aims to eliminate the complexity of air gap software delivery.  It does this by providing a way to package and deploy software in a way that is repeatable, secure, and reliable.

In this lab, you'll learn how to create a very basic Zarf package.

Creating a Zarf Package
===
Zarf can be used to package just about any type of software to run on a disconnected Kubernetes cluster. For this example we'll use the [ArgoCD Chart](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd) but the steps and tools used below are very similar for other applications.

Create the Package Definition
===
The most important piece of a Zarf package is the `zarf.yaml` file. It follows the [Zarf package schema](https://github.com/defenseunicorns/zarf/blob/main/zarf.schema.json) definition and allows us to specify the package metadata and the components we want to deploy.

In the code editor to the left, you should see an empty `zarf.yaml` file. Let's start by defining the `kind` and `metadata` of the Zarf package with the following content. Open the `zarf.yaml` file and type or copy the following content into it.
```yaml
kind: ZarfPackageConfig # ZarfPackageConfig is the package kind for most normal zarf packages
metadata:
  name: argocd       # specifies the name of our package and should be unique and unchanging through updates
  version: 9.4.4      # (optional) a version we can track as we release updates or publish to a registry
  description: |        # (optional) a human-readable description of the package that you are creating
    "A Zarf Package that deploys the ArgoCD platform"
```
Add the ArgoCD Baseline Component
===
In a Zarf package, components are the units that define an application stack. Components might be Kubernetes manifests, individual configuration files, Helm charts, or other items. They are specified under the `zarf.yaml` file under the `components` key and can include various resource types. For more details, see the [Understanding Zarf Components](https://docs.zarf.dev/ref/components/) page in the Zarf docs.

Let's add the following ArgoCD component to the bottom of our `zarf.yaml` below the initial metadata:

```yaml
components:
  - name: argocd # specifies the name of our component and should be unique and unchanging through updates
	  description: | # (optional) a human-readable description of the component you are defining
		  "Deploys the ArgoCD packaged chart into the cluster"
    required: true # (optional) sets the component as 'required' so that it is always deployed
    charts:
      - name: argo-cd
        version: 9.4.4
        namespace: argocd
        url: https://argoproj.github.io/argo-helm
        releaseName: argocd-baseline
        valuesFiles:
          - baseline-values.yaml
    images:
      - docker.io/library/redis:8.2.3-alpine
      - quay.io/argoproj/argocd:v3.3.2
```
Let's take a look at the last line of the components block. It specifies a `valuesFiles` array with a `baseline-values.yaml` item. This points to the the values files for the ArgoCD Helm chart that we're using for this package. You can read more about values files in the [Helm docs](https://helm.sh/docs/chart_template_guide/values_files/#helm). In the code editor, you should see the `baseline-values.yaml` file. open that file and add the following content:

```yaml
redis-ha:
  enabled: false

dex:
  enabled: false

notifications:
  enabled: false

redis:
  image:
    repository: docker.io/library/redis
```
> [!NOTE]
> We populate any `values.yaml` file(s) at this stage because the `zarf dev find-images` command we will use next will template out this chart to look only for the images we need. These values will be replaced later in the tutorial.

 Find the Images
===
Zarf is designed to help you easily run software in disconnected environments. One way it does this is by pulling all needed container images ahead of time and packaging them up along with everything else in your Zarf package.

Now that we've defined a Zarf package above, we can work on setting the images that we'll need to bring with us into the air gap. Zarf has a specific helper command for this. In the terminal window, go ahead and run:
```run
zarf dev find-images
```
Running this command in the same directory as your zarf.yaml should result in output similar to the following:

```yaml
components:
  - name: argocd
    images:
      - docker.io/library/redis:8.2.3-alpine
      - quay.io/argoproj/argocd:v3.3.2
      # Cosign artifacts for images - argocd
      - quay.io/argoproj/argocd:sha256-5882f28f7aaeaac397949c4511fdc1ad66c1260af44166ccf7e57aca3d7b9797.att
```
Copy the `images` key and array of images from your output into the `argo-cd` component we defined in our `zarf.yaml`

Set Up Variables
===
We now have a deployable package definition, but it is not very configurable. To make it more flexible, we need to add some variables to the `zarf.yaml` file. Add this content to the bottom of the file below everything else you've added up to this point:

TODO: do we want to include variables?

Zarf can template chart values, manifests, included text files and more.

> [!WARNING]
> When dealing with sensitive values in Zarf it is strongly recommended to not include them directly inside of a Zarf Package and to only define them at deploy-time. You should also be aware of where you are using these values as they may be printed in actions you create or files that you place on disk.

Set Up a Zarf Connect Service
===
As-is, our package could be configured to interface with an ingress provider to provide access to our argocd dashboard, but this may not be desired for every service, particularly those that provide a backend for other frontend services. To help with debugging, Zarf allows you to specify Zarf Connect Services that will be displayed after package deployment to quickly connect into our deployed application.

For this package we will leverage existing chart values to provide discoverable connect endpoints. These are normal Kubernetes services with special labels and annotations that Zarf watches for. Add the following content to the  `baseline-values.yaml` file:
```yaml

```

Create the Package
===
Once you have followed the above steps, you should now have a `zarf.yaml` file that looks very similar to the one found on the [Argocd example page](https://docs.zarf.dev/ref/examples/argocd/#zarfyaml)

Creating this package is as simple as running the `zarf package create` command within the same directory as our `zarf.yaml` file. Zarf will pull down all of the resources and bundle them into a package tarball.
```run
zarf package create .
```
This will create a zarf package in the current directory with a package name that looks something like `zarf-package-argocd-amd64-9.4.4.tar.zst`, although it might be slightly different depending on your system architecture. You can have a look at the file by listing the contents of the current directory:
```run
ls
```

Congratulations! You’ve built your very own Zarf  package! Zarf also builds a Software Bill of Materials (SBOM) into each package. In the next tutorial, you can learn how to inspect the SBOM in the package we just built.
