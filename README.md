# argocd
## Introduction
As part of our exploration into modern Kubernetes deployment practices, this Proof of Concept (POC) aims to demonstrate the setup and utilization of Argo CD, a powerful Continuous Delivery tool based on GitOps principles. 
Argo CD simplifies application deployments, configurations, and rollbacks in Kubernetes environments, allowing us to automate deployment processes effectively.

## What is Argo CD?
**Argo CD** is an open-source continuous delivery tool designed for Kubernetes clusters. 
It operates based on GitOps principles, enabling automation of deployment processes while maintaining the desired state of applications within the Kubernetes environment.

**GitOps** is a methodology that uses Git repositories as the source of truth for defining and managing infrastructure and application configurations. Argo CD leverages GitOps principles to ensure that the state of applications in Kubernetes matches the state defined in Git repositories, providing a reliable and auditable deployment process.

## How it works

Argo CD follows the **GitOps** pattern of using Git repositories as the source of truth for defining
the desired application state. Kubernetes manifests can be specified in several ways:

* [kustomize](https://kustomize.io) applications
* [helm](https://helm.sh) charts
* [jsonnet](https://jsonnet.org) files
* Plain directory of YAML/json manifests
* Any custom config management tool configured as a config management plugin

Argo CD automates the deployment of the desired application states in the specified target environments.
Application deployments can track updates to branches, tags, or pinned to a specific version of
manifests at a Git commit. See [tracking strategies](user-guide/tracking_strategies.md) for additional
details about the different tracking strategies available.


## Getting Started: Installing Argo CD on Kubernetes
1. Follow the official [Argo CD Documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/) to install Argo CD on the Kubernetes cluster.
2. Execute the installation commands provided in the documentation, ensuring to configure any specific settings required for our environment.
3. Verify the successful installation of Argo CD components using Kubernetes command-line tools (`kubectl`).
4. This default installation will have a self-signed certificate and cannot be accessed without a bit of extra work.
Do one of:

* Follow the [instructions to configure a certificate](https://argo-cd.readthedocs.io/en/stable/operator-manual/tls/) .
* Configure the client OS to trust the self signed certificate.


## Accessing the Argo CD Web UI
1. Retrieve the external IP or domain of the Argo CD server using the `kubectl get svc` command.
2. Access the Argo CD Web UI via a web browser using the obtained IP or domain.
3. Authenticate into the Argo CD UI using the default credentials or any configured authentication method.

## Setting Up the Git Repository
1. Create a Git repository to store Kubernetes manifests for applications.
2. Organize Kubernetes manifests within the repository, adhering to GitOps principles.
3. Ensure the repository is accessible and writable by Argo CD for synchronization purposes.

## Creating an Argo CD Application
1. Craft an Argo CD Application YAML file to define the application's source, destination, and other configurations.
2. Replace placeholders in the YAML file with actual values such as repository URL, target namespace, and project name.
3. Apply the Argo CD Application configuration using `kubectl apply -f <application.yaml>`.

## Observing Application Deployment
1. Monitor the Argo CD UI for real-time progress updates on application synchronization.
2. Ensure that the application deployment progresses without errors, indicating successful synchronization with the Git repository.

## Rollback and Declarative Updates
1. Modify the Argo CD Application YAML file to trigger a declarative update.
2. Observe Argo CD's ability to rollback to previous versions in case of configuration changes or failures.
3. Verify the effectiveness of rollback functionality by confirming the application state matches the desired state defined in the Git repository.

## Conclusion
In conclusion, this POC demonstrates the simplicity and effectiveness of using Argo CD for Kubernetes deployments. By embracing GitOps principles, we achieve automation, synchronization, and rollback capabilities, enhancing the reliability and consistency of our deployment processes.

## Fix redirection issues!
To fix redirection issues We need to disable internal TLS in the ArgoCD server and , we need to modify the command line arguments passed to the argocd-server container in the deployment,  the argument --insecure needs to be added to the command line arguments. 

 	kubectl -n argocd edit deployment.apps argocd-server

	- Verify that the ARGOCD_SERVER_INSECURE environment variable is correctly set to true. In your deployment YAML, you have:
	
	```yaml
	- name: ARGOCD_SERVER_INSECURE
	  valueFrom:
	    configMapKeyRef:
	      key: server.insecure
	      name: argocd-cmd-params-cm
	      optional: true
        ```  

Modify the args field to include --insecure in the command line arguments:

```yaml
          - /usr/local/bin/argocd-server
	  - --insecure  # Add this line
``` 
	

Log in to the ArgoCD web interface https://argocd.acp.cloud.atos.net/ by using the default username admin and the password, collected by the following command.
```bash
	kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"|base64 -d
``` 

```bash
	$base64Password = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
	[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($base64Password))
```


