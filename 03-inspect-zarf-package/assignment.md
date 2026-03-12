---
slug: inspect-zarf-package
id: 8kahsfqxpzxj
type: challenge
title: Inspect a Zarf Package
teaser: Understand the composition of a Zarf Package on the filesystem and deployed
  to a cluster
tabs:
- id: ffiresxmfuq1
  title: Terminal
  type: terminal
  hostname: kubernetes-vm
  workdir: /root/zarf-package
difficulty: basic
timelimit: 0
lab_config:
  default_layout_sidebar_size: 0
enhanced_loading: false
---
Inspecting a Zarf Package
===
A Zarf Package is meant to be a transparent "envelope" of what is packaged for secure software delivery. This is intentional such that they can transition organizational boundaries with provenance and trust.

The location of a Zarf Package for inspection can be in multiple primary locations,

You can inspect:
- A Zarf Package tarball on your filesystem
- A Zarf Package published to an OCI Registry
- A Zarf Package deployed to your Kubernetes Cluster

Declarative Packaging and the filesystem (optional)
===
You might be asking yourself - "What actually happened earlier when I packaged ArgoCD?"

Let's take a look:
```run
zarf tools archiver decompress zarf-package-argocd-amd64-9.4.4.tar.zst unarchived/ --unarchive-all
```
What you will see should look like the following:
```bash
unarchived
├── checksums.txt
├── components
│   └── argocd
│       ├── charts
│       │   └── argo-cd-9.4.4.tgz
│       └── values
│           └── argo-cd-9.4.4-0
├── images
│   ├── blobs
│   │   └── sha256
│   │       ├── 14d29e4d8fcbbd300ed3e6e921741d935ac4f559ed524e8b66982bdbb5380076
│   │       ├── 14fe09893f20eae12c88a85d0a5eaa0224235dbdd3654fbc8be91bc5e8986a99
│   │       ├── 2894fcdf95d060b9ec10eda4fff99cf11b9d18fb344e8bd6051ed44e39b5f80a
│   │       ├── 2f25135cc937748dc7f12ddd61abe8c7cc1f5eedc0322b8a68a7816b40f885d3
│   │       ├── 2fe148588d93987cea3533873195ec2450f5dbf420c78606aaacad7b78bb5489
│   │       ├── 31e08567dbea7eb0906faac4261e2436448d8d3e0bc606e5d31c44a181aa51e5
│   │       ├── 3bf6d9d5eca23eec7f7429505817bda0f9c88fb3bf8a0bebe79409fc3aafba94
│   │       ├── 4181ccaa7e4d1c04c6a0719984e7174ed25f17a4125ce6528a1f0892b930412e
│   │       ├── 41ec20fa8059b65bf31b5cc81231e83b9167a403dad325ba34d11f6a66eeeb9e
│   │       ├── 432af7f450c1e87b133470782ea76f04e715f0b31aa1ec59dc9bec2e333362f6
│   │       ├── 4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1
│   │       ├── 5cdcdab8682dd701898a0b32f30f63a7279955d6579534e3db2236490fbcc659
│   │       ├── 5e4e3926b2b2633c855cf8585973d5c4f4d24653a143c4279ee4f069528aff22
│   │       ├── 6a0b9373c91367ca35262a721578167a510e69e6610997235b9cc2192a07e8bd
│   │       ├── 6b1e74f2df0023cba95dc7acc9a9ca8e32a30387fcf775d9b26e54c828dd7d42
│   │       ├── 73d9383a4783041b53631dcc5abf5172254459b430cad4a305be2d0edcc0de7c
│   │       ├── 76a706c2758d73147a034dd88577d991a35dda407c36cbf75d24ecc9d8601ebb
│   │       ├── 7dc4da3a2de931d1c82d12b569038c90df13fe90be2b87d807201b133b4b4d21
│   │       ├── aeee62fd89e5adc25112672830e491a565a29086fc4562f4fe730212ffc89b33
│   │       ├── c1376ce6ec799c41153bd8532007a952793e2f9c90e86401ee036d66e5612dec
│   │       ├── ca1b97d7dd35cc1607cd75a0370772cc6e6287cc6636b2de723a58b7dc33a869
│   │       ├── d741ee1608f399e21c72d05f0f818c348c6801af33aeb83523893d09dc153957
│   │       ├── d818fdb0c1daf05988901e08684f1f08fdebeee4e4effb81a308f69f6c11cb60
│   │       ├── e914881e1e93c7f1edcee71d527c2800fbc0c4964fb338c063b63805d91f8040
│   │       └── fb81e3e80963fbd490c3dcfbbaedd1125f8f9d22cc3fa832444b9bb4fc3f5322
│   ├── index.json
│   ├── ingest
│   └── oci-layout
├── sboms
│   ├── compare.html
│   ├── docker.io_library_redis_8.2.3-alpine.json
│   ├── quay.io_argoproj_argocd_v3.3.2.json
│   ├── sbom-viewer-docker.io_library_redis_8.2.3-alpine.html
│   └── sbom-viewer-quay.io_argoproj_argocd_v3.3.2.html
└── zarf.yaml
```
This expands as you add more components to the manifest in such a way that Zarf can deterministically deploy 1->N applications from a given manifest.

If we were to sign this Zarf Package - you would additionally see the signature included in the archive - creating more portable provenance for cryptographic integrity.

Inspect Commands (filesystem)
===
The `zarf package inspect` commands provide further transparency into the package.
```run
zarf package inspect --help
```
Will output the following sub-commands:
```bash
  definition    Displays the 'zarf.yaml' definition for the specified package
  documentation Extract documentation files from the package
  images        List all container images contained in the package
  manifests     Template and output all manifests and charts in a package
  sbom          Output the package SBOM (Software Bill Of Materials) to the specified directory
  values-files  Creates, templates, and outputs the values-files to be sent to each chart
```

For example - if you want to see the `zarf.yaml` definition for a given package:
```run
zarf package inspect definition zarf-package-argocd-amd64-9.4.4.tar.zst
```

To see all of the rendered manifests for the packaged helm charts and manifests:
```run
zarf package inspect manifests zarf-package-argocd-amd64-9.4.4.tar.zst
```

Similarly the `values-files`
```run
zarf package inspect values-files zarf-package-argocd-amd64-9.4.4.tar.zst
```

To get an local directory of the package sboms
```run
zarf package inspect sbom zarf-package-argocd-amd64-9.4.4.tar.zst
```

The images you included in the manifest for quick reference:
```run
zarf package inspect images  zarf-package-argocd-amd64-9.4.4.tar.zst
```

Not shown - but if you had included documentation in the package it could be retrieved as well.

Inspect a Deployed Package
===
Zarf is not only packaging applications into determinstic archives - it also performs the deploy as we performed previously. It does so while keeping track of the state. Enabling users to identify which versions of application(s) they have deployed as well as inspecting or removing them.

To inspect deploy packages, we can first list them:
```run
zarf package list
```

From here, we can use a subset of the inspect commands currently available.
```run
zarf package inspect definition argocd
```
Will output the zarf definition (manifest) of the deployed package.

If we want to see our Images from a package
```run
zarf package inspect images argocd
```

Note: currently Zarf does not store SBOMs or values-files for the deployed packages.

Remove a Package from the Cluster
===
Given that we store the deployed package state, we have the option to continue to upgrade those package applications in-place as well as remove a package and all of its applications.

Confirm the `argocd` package is still deployed:
```run
zarf package list
```

To remove the `argocd` package from the cluster:
```run
zarf package remove argocd
```

This will prompt you to remove the package - otherwise you can use the `--confirm` flag to auto-confirm removal.




