# Cyclops GitOps starter

GitOps is a practice of storing configuration and infrastructure definitions in a git repository. The same way developers store application code on git, GitOps extends that practice to application deployment. There are tools that marry the GitOps methodology with Kubernetes, like ArgoCD, Flux, Jenkins X…

This repo serves as a tutorial on how to integrate Cyclops with GitOps, namely ArgoCD.

Feel free to fork the repo, play around with it, and propose alternative approaches! 🧡

## Prerequisites

Here are the prerequisites for the rest of the tutorial and how to set them up.

- Kubernetes cluster
    - if you don't have a running cluster you can use, you can install and run [minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download)
- [kubectl](https://pwittrock.github.io/docs/tasks/tools/install-kubectl/)
- ArgoCD
    - [tutorial on setting up ArgoCD and exposing it outside of the cluster](https://medium.com/@mehmetodabashi/installing-argocd-on-minikube-and-deploying-a-test-application-caa68ec55fbf)
    - [official docs](https://argo-cd.readthedocs.io/en/stable/#getting-started)
- Cyclops
    - [how to install](https://cyclops-ui.com/docs/installation/install/manifest)
- Clone this repository

## 👁️ Cyclops x ArgoCD 🦑

All applications within Cyclops are defined as CRDs called Modules. Each time a Module is created, the Cyclops controller picks it up, creates other Kubernetes resources from it, and applies them to the cluster. Since Modules are CRDs, you can define them via the YAML manifest. Those manifests allow you to define everything you need for your application in a single place.

Since a Module can be defined through a manifest, it can be stored on a git repo and included in your GitOps workflow! In this repository, you can find the `argo-app.yaml` file and the `/apps` folder that holds module definitions.

The picture below shows the hierarchy of resources inside the cluster with the setup.

![app hierarchy](https://github.com/user-attachments/assets/e0716c9e-bcc9-4312-a6a9-e34fc867e463)


In the root of the hierarchy, there is the `my-team-apps` Argo application defined in the `argo-app.yaml` file in the root of the repo. The application deploy everything from the `https://github.com/cyclops-ui/gitops-starter` repo (this one) under path `apps`.

As mentioned, that folder contains three module definitions separated into respective files. When the ArgoCD controller syncs those Modules into the cluster, the Cyclops controller takes over and turns those Modules into other Kubernetes resources like Deployment, Services, StatefulSets, or whatever is defined in the Helm chart referenced by the Module and its values.

### See it in action 🔥

To deploy the proposed setup from above, ensure all of the prerequisites are met and you can access your ArgoCD and Cyclops instances.

Open your terminal and `cd` to this repository. You can create the `my-team-apps` Argo application with the following command:

```bash
kubectl apply -f argo-app.yaml
```

You can now see your newly created Argo application in your ArgoCD UI:

<img width="1506" alt="argo app" src="https://github.com/user-attachments/assets/f70e045b-bc8f-44aa-ad5a-e0dbf39dbcc4">

All of the Modules are also visible in the Cyclops UI:

<img width="1503" alt="cyclops modules" src="https://github.com/user-attachments/assets/809d8fb9-9a4a-4264-a8a0-610d596b032a">

You can now check all of the resources created from a Module through the Cyclops UI. You are also free to update app configuration through Cyclops.

When updating a Module through Cyclops, you could introduce a diff in ArgoCD. For example, while updating replicas through Cyclops, Argo might report it as in the image below

<img width="1507" alt="argo diff" src="https://github.com/user-attachments/assets/a41bcfc9-7f96-4055-b88a-012e4ba5d751">

To avoid this, you can simply **remove the field** you want to edit through the UI from the **Module manifest** in git.

If you want to restrict edits of any fields explicitly, you can mark them as `immutable` in your helm chart schema. An example of a template with immutable fields is used for the `other-app` you have already deployed.

<img width="1509" alt="immutable" src="https://github.com/user-attachments/assets/c013738b-cd4d-48bc-b53d-4e936cf6f640">

In the image above is the edit screen of the `other-app`, and fields for image and version cannot be updated and should be updated through the GitOps workflow.

## Roadmap 🚧

> *⚠️ Please note, our roadmap is subject to change. ⚠️*
>
>
> *If there's something you'd like to see added, feel free to reach out and let us know!*
>
- Option to disable the ***Add module*** button
    - in case you want to restrict creating a Module to your GitOps workflow
- Option to disable changing the template in the Edit template screen
    - in case you want to restrict updating Module templates to your GitOps workflow
