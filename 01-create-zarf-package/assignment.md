---
slug: create-zarf-package
id: haa2sqszamql
type: challenge
title: Create a Zarf Package
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
---
Getting Started with Zarf
===
![zarf.png](https://play.instruqt.com/assets/tracks/ik0jeigqmuwc/cca3978f9217d4cfdbcf6f22c81d0d43/assets/zarf.png)

Zarf is an open source software packaging tool that aims to eliminate the complexity of air gap software delivery.  It does this by providing a way to package and deploy software in a way that is repeatable, secure, and reliable.

In this lab, you'll learn how to create and a very basic Zarf package.

Creating a Zarf Package
===
Zarf can be used to package just about any type of software to run on a disconnected Kubernetes cluster. For this example we'll use the [WordPress chart from Bitnami](https://artifacthub.io/packages/helm/bitnami/wordpress) but the steps and tools used below are very similar for other applications.

Create the Package Definition
===
The most important piece of a Zarf package is the `zarf.yaml` file. It follows the [Zarf package schema](https://github.com/defenseunicorns/zarf/blob/main/zarf.schema.json) definition and allows us to specify the package metadata and the components we want to deploy.

In the code editor to the left, you should see an empty `zarf.yaml` file. Let's start by defining the `kind` and `metadata` of the Zarf package with the following content. Open the `zarf.yaml` file and type or copy the following content into it.
```yaml
kind: ZarfPackageConfig # ZarfPackageConfig is the package kind for most normal zarf packages
metadata:
  name: wordpress       # specifies the name of our package and should be unique and unchanging through updates
  version: 16.0.4       # (optional) a version we can track as we release updates or publish to a registry
  description: |        # (optional) a human-readable description of the package that you are creating
    "A Zarf Package that deploys the WordPress blogging and content management platform"
```
Add the WordPress Component
===
In a Zarf package, components are the units that define an application stack. Components might be Kubernetes manifests, individual configuration files, Helm charts, or other items. They are specified under the `zarf.yaml` file under the `components` key and can include various resource types. For more details, see the [Understanding Zarf Components](https://docs.zarf.dev/ref/components/) page in the Zarf docs.

Let's add the following WordPress component to the bottom of our `zarf.yaml` below the initial metadata:

```yaml
components:
  - name: wordpress  # specifies the name of our component and should be unique and unchanging through updates
    description: |   # (optional) a human-readable description of the component you are defining
      "Deploys the Bitnami-packaged WordPress chart into the cluster"
    required: true   # (optional) sets the component as 'required' so that it is always deployed
    charts:
      - name: wordpress
        url: oci://registry-1.docker.io/bitnamicharts/wordpress
        version: 16.0.4
        namespace: wordpress
        valuesFiles:
          - wordpress-values.yaml
```
Let's take a look at the last line of the components block. It specifies a `valuesFiles` array with a `wordpress-values.yaml` item. This points to the the values files for the WordPress Helm chart that we're using for this package. You can read more about values files in the [Helm docs](https://helm.sh/docs/chart_template_guide/values_files/#helm). In the code editor, you should see the `wordpress-values.yaml` file. open that file and add the following content:

```yaml
wordpressUsername: zarf
wordpressPassword: ""
wordpressEmail: hello@defenseunicorns.com
wordpressFirstName: Zarf
wordpressLastName: The Axolotl
wordpressBlogName: The Zarf Blog

# This value turns on the metrics exporter and thus will require another image.
metrics:
  enabled: true

# Sets the WordPress service as a ClusterIP service to not conflict with potential
# pre-existing LoadBalancer services.
service:
  type: ClusterIP
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

  - name: wordpress
    images:
      - docker.io/bitnami/apache-exporter:0.13.3-debian-11-r2
      - docker.io/bitnami/mariadb:10.11.2-debian-11-r21
      - docker.io/bitnami/wordpress:6.2.0-debian-11-r18
```
Copy the `images` key and array of images from your output into the `wordpress` component we defined in our `zarf.yaml`

Set Up Variables
===
We now have a deployable package definition, but it is not very configurable. To make it more flexible, we need to add some variables to the `zarf.yaml` file. Add this content to the bottom of the file below everything else you've added up to this point:

```yaml
variables:
  - name: WORDPRESS_USERNAME
    description: The username for the WordPress admin account
    default: zarf
    prompt: true
  - name: WORDPRESS_PASSWORD
    description: The password for the WordPress admin account
    prompt: true
    sensitive: true
  - name: WORDPRESS_EMAIL
    description: The email for the WordPress admin account
    default: hello@defenseunicorns.com
    prompt: true
  - name: WORDPRESS_FIRST_NAME
    description: The first name for the WordPress admin account
    default: Zarf
    prompt: true
  - name: WORDPRESS_LAST_NAME
    description: The last name for the WordPress admin account
    default: The Axolotl
    prompt: true
  - name: WORDPRESS_BLOG_NAME
    description: The blog name for the WordPress site
    default: The Zarf Blog
    prompt: true
```
To use these variables in our chart we must add their corresponding templates to our wordpress-values.yaml file. You can replace the existing variables with these values:
```yaml
wordpressUsername: ###ZARF_VAR_WORDPRESS_USERNAME###
wordpressPassword: ###ZARF_VAR_WORDPRESS_PASSWORD###
wordpressEmail: ###ZARF_VAR_WORDPRESS_EMAIL###
wordpressFirstName: ###ZARF_VAR_WORDPRESS_FIRST_NAME###
wordpressLastName: ###ZARF_VAR_WORDPRESS_LAST_NAME###
wordpressBlogName: ###ZARF_VAR_WORDPRESS_BLOG_NAME###
```
Zarf can template chart values, manifests, included text files and more.

> [!WARNING]
> When dealing with sensitive values in Zarf it is strongly recommended to not include them directly inside of a Zarf Package and to only define them at deploy-time. You should also be aware of where you are using these values as they may be printed in actions you create or files that you place on disk.

Set Up a Zarf Connect Service
===
As-is, our package could be configured to interface with an ingress provider to provide access to our blog, but this may not be desired for every service, particularly those that provide a backend for other frontend services. To help with debugging, Zarf allows you to specify Zarf Connect Services that will be displayed after package deployment to quickly connect into our deployed application.

For this package we will define two services, one for the blog and the other for the WordPress admin panel. These are normal Kubernetes services with special labels and annotations that Zarf watches for. Add the following content to the  `connect-services.yaml` file:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-connect-blog
  labels:
    # Enables "zarf connect wordpress-blog"
    zarf.dev/connect-name: wordpress-blog
  annotations:
    zarf.dev/connect-description: "The public facing WordPress blog site"
spec:
  selector:
    app.kubernetes.io/instance: wordpress
    app.kubernetes.io/name: wordpress
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-connect-admin
  labels:
    # Enables "zarf connect wordpress-admin"
    zarf.dev/connect-name: wordpress-admin
  annotations:
    zarf.dev/connect-description: "The login page for the WordPress admin panel"
    # Sets a URL-suffix to automatically navigate to in the browser
    zarf.dev/connect-url: "/wp-admin"
spec:
  selector:
    app.kubernetes.io/instance: wordpress
    app.kubernetes.io/name: wordpress
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
```

We also need to make the Zarf package aware of these services. To do that, add the following content nested under the existing `wordpress` component in the `zarf.yaml` file:
```yaml
    manifests:
      - name: connect-services
        namespace: wordpress
        files:
          - connect-services.yaml
```

Create the Package
===
Once you have followed the above steps, you should now have a `zarf.yaml` file that looks very similar to the one found on the [WordPress example page](https://docs.zarf.dev/ref/examples/wordpress/#zarfyaml).

Creating this package is as simple as running the `zarf package create` command within the same directory as our `zarf.yaml` file. Zarf will show us the `zarf.yaml` one last time asking if we would like to build the package, and upon confirmation Zarf will pull down all of the resources and bundle them into a package tarball.
```run
zarf package create .
```
This will create a zarf package in the current directory with a package name that looks something like `zarf-package-wordpress-amd64-16.0.4.tar.zst`, although it might be slightly different depending on your system architecture. You can have a look at the file by listing the contents of the current directory:
```run
ls
```

Congratulations! You’ve built your very own Zarf  package! Zarf also builds a Software Bill of Materials (SBOM) into each package. In the next tutorial, you can learn how to inspect the SBOM in the package we just built.
