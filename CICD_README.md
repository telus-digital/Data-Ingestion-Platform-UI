# CCOE CI/CD Generator

Thank you for using our [simplify starter kit]('https://simplify.telus.com/create/templates/default/ccoe-canary-cicd-generator?lang=en') to bootstrap your CICD pipeline! The files contained in this PR are designed to manage different aspects of the CI/CD pipeline for the application, and collectively enable a robust CI/CD pipeline, providing capabilities for initialization, deployment, and rollback. Remember to pay attention to the any error messages, use online resources to debug, and, if all else fails, feel free to search & ask for help in the [canary-cicd-support channel](https://telus.enterprise.slack.com/archives/C08RMFR7ME3).  

### Prerequisites (Setting up GitHub Environments)
This workflow interacts heavily with your GitHub Repo's Environments (**Settings | Environments**.). In particular, for each environment you are deploying to, **you MUST HAVE a GitHub `environment` of the same name created in your repository's settings**. Additionally, if you would like to set up manual approval checkpoints to protect deployments to certain environments, you can seup `protection rules`. Here's how you can set that up:
1. In your GitHub repository, navigate to **Settings | Environments**.. (If you cannot see either tab, ensure you have the necessary permissions for your repository).
2. Click `New Environment`, and configure an environment with the same name as you are using as target environments in your workflow.
3. Select **Configure/Edit Environment**.
4. Under **Deployment Protectection Rules**, select the **Required Reviews** rule, and add github users/teams you'd like to assign to be able to approve deployments for the selected environment.
5. Click **Save Protection Rule**. The next time you run the pipeline, it will require manual approval before deploying to the configured environment(s).

You are now ready to run the workflow to deploy to securely deploy to your environments.

### Wask Template Compatibility:
1. Review our [quick start guide](https://simplify.telus.com/docs/cloud-coe-documentation/lawnmower/QuickStart.md) to ensure you have all the prequisites (Onboarded GCP Project, GitHub Repo WIF, etc), and the section in the [quickstart_FAQ question](https://simplify.telus.com/docs/cloud-coe-documentation/lawnmower/QuickStart_FAQ.md?lang=en#how-can-i-get-the-reusable-workflows-to-work-with-the-wask-starter-kit). 
---

## CICD Pipeline component summary:

### **`cd-deploy.yaml`**

#### **Purpose**:

A GitHub Action workflow is a configurable, automated process that executes one or more scripts or actions. They are defined by yaml files, and are checked into your repository (`.github/workflow/`), where they can be triggered by specified events.

This workflow manages a deployment pipeline that rolls out resources to our GKE cluster using Google Cloud Deploy. It builds the necessary artifacts and deploys them to the specified environments. The workflow can eiher be a completely custom implementation, or facilitated using a pair of [reusable GHA workflows]('https://github.com/telus/ccoe-reusable-workflows') managed by the CCoE (please see that repo for a complete and up-to-date list of available inputs and behaviours).

Very broadly, our standard CICD pipelines that deploy resources into our shared GKE clusters will contain the following main steps:
1. **Build**: Compile source code, build any necessary artifacts (like container images, or cloud resources), and upload them to a remote location.
2. **Deploy**: Deploy the resources built in step 1 by rendering our configuration files, and/or pulling in pre-build artifacts to he desired environments.
3. **Vaildate, monitor & test**: Validate that the deployed resources behave as expected, and monitor them for errors. Rollback if necessary.

---

### **`cd-rollback.yaml`**

#### **Purpose**:
A GitHub Action workflow is a configurable, automated process that executes one or more scripts or actions. They are defined by yaml files, and are checked into your repository (`.github/workflow/`), where they can be triggered by specified events.

This workflow provides a manual rollback mechanism for Cloud Deploy. It allows users to roll back to the last successful deployment for a specified target Google Cloud Deploy environment.

---

### **`Helm values`**

#### **Purpose**:

Helm is a templating tool that allows teams to easily render complete manifests by passing in `Values` file(s) to be built using `templates`. The templates are writen, and uploaded to remote locations as `Charts`. At Telus, there are two major charts available in our remote artifact registry:

- [Our Standard / CIO chart](https://github.com/telus/cdo-gke-charts/tree/main/cio-gke-deployment) provides teams with a simple way to create resources ranging from cronjobs, to fully supported service-mesh deployments. Teams are **highly encouraged** to use this chart for their deployments to remain up to date with the recommended configuration from the CCoE.
- [Our Legacy / Digital chart](https://github.com/telus/digital-gke-deployment-chart/tree/main) is available to provide compatibility for teams who are managing a `shippy` application in the td-digital clusters. Teams are free to use this chart, but are **highly encouraged** to use our standard chart instead, as this chart creates unnecessary resources, and does not provide the same features and security as our standard chart. 

Each environment should have its own distinct helm `values` file that is used to build the specific resources for that environment.

---

### **`Dockerfile`**

#### **Purpose**:

A Dockerfile is a text document that contains a series of instructions that define the steps required to create an image, which is  used to run containers. Dockerfiles enable repeatable builds and ensure consistency across development, testing, and production environments. Once built, the resulting image can be deployed anywhere Docker is supported, including cloud platforms like Google Kubernetes Engine (GKE).

In general, your deployment pipeline should build a container image, and upload it to a remote repository where it can be pulled later by your deployed GKE resources

Container/Dockerfiles can be tested and ran locally using free tools like Docker (CLI only), podman Desktop or Rancher.

---

### **`CloudDeploy.yaml`**

#### **Purpose**:

Defines the deployment pipeline configuration for Google Cloud Deploy. This file specifies the deployment targets, release strategies, and approval processes for your application. It acts as the blueprint for managing progressive rollouts, ensuring safe and controlled deployments across multiple environments.

---

### **`Skaffold.yaml`**

#### **Purpose**:

Provides configuration for building, tagging, and deploying your application using Skaffold. This file simplifies the development and deployment process by automating workflows, managing Kubernetes manifests, and integrating with CI/CD pipelines. It ensures consistency and efficiency in deploying containerized applications.

---
