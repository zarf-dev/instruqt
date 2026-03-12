---
slug: package-playground
type: challenge
title: Zarf Package Playground
teaser: Playground environment for self-exploration of package creation
tabs:
- title: Terminal
  type: terminal
  hostname: kubernetes-vm
  workdir: /root/zarf-package
- title: Editor
  type: code
  hostname: kubernetes-vm
  path: /root/zarf-package
difficulty: basic
timelimit: 0
enhanced_loading: false
---
Welcome! If you've made it this far you have now:
- Created a Zarf Package for ArgoCD
- Deployed the ArgoCD Package to a Kubernetes cluster
- Inspected the Zarf Package in your local filesystem and as deployed to the cluster
- Removed the ArgoCD Package from the cluster

Now you are free to create Zarf Packages for your applications.

Package Playground
===

This Package playground includes:
- The Zarf binary
- A Kubernetes Cluster
- A skeleton `zarf.yaml` - Just fill in your helm chart and images!

Feel free to explore popular applications and see the [Zarf Examples](https://docs.zarf.dev/ref/examples/) for more inspiration.
