---
slug: deploy-zarf-package
id: q5pvoy9brjr0
type: challenge
title: Deploy a Zarf Package
tabs:
- id: tnnkmegkjyre
  title: Terminal
  type: terminal
  hostname: zarf-vm
  workdir: /root/zarf-package
- id: 1lywpqijrhot
  title: Application
  type: service
  hostname: zarf-vm
  port: 42000
difficulty: ""
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
---
Deploy a Zarf Package
===

In the last lab, you learned how to create a Zarf package. Now, you'll learn how to deploy that package onto a local air-gapped Kubernetes cluster. Let's get started!


Inspect the Environment
===

We've only got a terminal window this time around, but you can see the Zarf package built in the last tutorial by listing the contents of the directory:
```run
ls
```
There is also a local Kubernetes cluster running using [K3s](https://k3s.io). Zarf includes K3s as an optional helper component that can be used when deploying to single air-gapped devices. You can learn more about the optional components in the [Zarf docs](https://docs.zarf.dev/ref/init-package/#optional-components). To inspect the details of the local Kubernetes cluster, you can run:
```run
kubectl cluster-info
```


Deploy the Package
===

Let's run the `zarf package deploy` command to kick off the deployment. You will be prompted to enter values for the variables we defined when creating the packge. You can stick to the default values for everything but the password value.
```run
zarf package deploy
```
To have a look at what Zarf deployed as part of the package, you can run:
```run
kubectl get all -n wordpress
```
Here you can see the various elements that were part of the WordPress Helm chart that we included as a component in our Zarf package.


View the WordPress Blog
===

As you might have previously read, Zarf is designed to work in environments with unique connectivity restrictions. In this case, we don't have any immediate way to access the application package we just deployed. To do this, we'll setup a port forward using `kubectl`:
```run
kubectl port-forward -n wordpress service/wordpress-connect-blog 42000:8080 --address 0.0.0.0
```
Once this command is run and the port forward is started, you should be able to navigate to the `Application` tab, and see the example blog.


Conclusion
===

Congratulations! You've now built and depolyed a simple Zarf package! There's a lot more that Zarf can do from here to help make software delivery into air-gapped environments easier. To see some more examples, check out [this folder](https://github.com/zarf-dev/zarf/tree/main/examples) in the Zarf repo. You can also learn more by reading through the [Zarf docs](https://docs.zarf.dev).
