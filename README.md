<a href="https://www.instagram.com/ttomasferrari/">
    <img align="right" alt="Abhishek's Instagram" width="22px" 
    src="https://i.imgur.com/EzpyGdV.png" />
</a>
<a href="https://twitter.com/tomasferrari">
    <img align="right" alt="Abhishek Naidu | Twitter" width="22px"         
    src="https://i.imgur.com/eFVBTVz.png" />
</a>
<a href="https://www.linkedin.com/in/tomas-ferrari-devops/">
    <img align="right" alt="Abhishek's LinkedIN" width="22px" 
    src="https://i.imgur.com/pMzVPqj.png" />
</a>
<p align="right">
    <a >Find me here: </a>
</p>
<!-- <p align="right">
    <a  href="/docs/readme_es.md">Versión en Español</a>
</p> -->

<p title="All The Things" align="center"> <img src="https://i.imgur.com/0iRDtob.jpg"> </p>

# **INSANE EDITION**

This Insane Edition builds upon the [Hardcore Edition](https://github.com/tferrari92/automate-all-the-things-hardcore).

### New features:

- Prometheus Operator: We'll be needing to use PodMonitors and ServiceMonitors, so we'll deploy the kube-prometheus-stack Helm chart which also includes the Prometheus operator, instead of just plain Prometheus.
- Service mesh with Istio
- Ingress Gateway with Istio
- Canary deployments with Flagger
  - Flagger Load Tester included
- Service mesh visualization with Kiali

### Versions in order of complexity:

1. [Regular Edition](https://github.com/tferrari92/automate-all-the-things)
2. [Hardcore Edition](https://github.com/tferrari92/automate-all-the-things-hardcore)
3. [Insane Edition](https://github.com/tferrari92/automate-all-the-things-insane)

<br/>

# **INDEX**

- [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [What we'll be doing](#what-well-be-doing)
  - [Tools we'll be using](#tools-well-be-using)
  - [Disclaimer](#disclaimer)
- [Local Setup](#local-setup)
- [Azure DevOps Setup](#azure-devops-setup)
  - [Create project](#create-project)
  - [Install required plugins](#install-required-plugins)
  - [Get your AWS keys](#get-your-aws-keys)
  - [Create AWS service connection](#create-aws-service-connection)
  - [Create DockerHub service connection](#create-dockerhub-service-connection)
  - [Create AWS-keys variable group](#create-aws-keys-variable-group)
  - [Create an Azure self-hosted agent](#optional-create-an-azure-self-hosted-agent)
- [AWS Infrastructure Deployment Pipeline](#aws-infrastructure-deployment-pipeline)
  - [Description](#description)
  - [Instructions](#instructions)
- [Backend Service Build & Deploy Pipeline](#backend-service-build--deploy-pipeline)
  - [Description](#description-1)
  - [Instructions](#instructions-1)
- [Frontend Service Build & Deploy Pipeline](#frontend-service-build--deploy-pipeline)
  - [Description](#description-2)
  - [Instructions](#instructions-2)
- [ArgoCD Deployment Pipeline](#argocd-deployment-pipeline)
  - [Description](#description-3)
  - [Instructions](#instructions-3)
- [Kubernetes Tools Management](#kubernetes-tools-management)
  - [Description](#description-4)
  - [Instructions](#instructions-4)
- [A Little About Istio](#a-little-about-istio)
  - [And what the hell is Flagger?](#and-what-the-hell-is-flagger)
  - [Other details](#other-details)
- [Destroy All The Things Pipeline](#destroy-all-the-things-pipeline)
  - [Description](#description-5)
  - [Instructions](#instructions-5)
- [Conclusion](#conclusion)
  - [On the next edition](#on-the-next-edition)

<br/>

# **INTRODUCTION**

I believe in a world where all that's expected of me is to enjoy life, lay on the couch, play COD and have existential crises.

I wish I could automate cooking, cleaning, working, doing taxes, making friends, dating and even writing stupid READMEs.

But technology hasn't quite caught up to my level of laziness yet, so I've taken some inspiration from Thanos and said ["Fine... I'll do it myself"](https://www.youtube.com/watch?v=EzWNBmjyv7Y).

Here's my attempt at making the world a better place. People in the future will look back at heroes like me and enjoy their time playing video games and fighting the war against AI, in peace.

<br/>

## Prerequisites

- [Git installed](https://www.python.org/downloads/)
- [Python3 installed](https://www.python.org/downloads/)
- [Active GitHub account](https://github.com/)
- [Active DockerHub account](https://hub.docker.com/)
- [Active AWS account](https://aws.amazon.com/)
- [Active Azure DevOps account](https://azure.microsoft.com/en-us/free/)

<br/>

## What we'll be doing

<p title="Diagram" align="center"> <img img width="800" src="https://i.imgur.com/HeuvWw9.jpg"> </p>

<br/>

The purpose of this repo is not to give you an in depth explanation of the tools we'll be using, but to demonstrate how they can interact with each other to make the deployment of a whole infrastructure (with an application) as efficient and streamlined as possible.

I want to show how IaC (Infrastructure as Code), Gitops and CI/CD (Continuous Integration/Continuous Deployment) can be merged for [unlimited power](https://www.youtube.com/watch?v=Sg14jNbBb-8).

As you can see in the diagram, we'll be deploying an EKS Kubernetes cluster in AWS. Inside the cluster we'll have three environments where our app will be deployed. The app is made up of two microservices: frontend and backend. Each frontend will be accesible to the public internet through a Load Balancer.

Along with the cluster, we'll deploy an ElastiCache (Redis) database for each environment and one EC2 instance for running configuration tasks on the databases. We'll also create an S3 bucket which will store our terraform state file.

Our app is a very simple static website, but I'm not spoiling it for you. You'll have to deploy it to see it.

<br/>

## Tools we'll be using

- Code Versioning -> Git
- Source Code Management -> GitHub
- Cloud Infrastructure -> Amazon Web Services
- Infrastructure as Code -> Terraform
- Containerization -> Docker
- Container Orchestration -> Kubernetes
- Continuous Integration -> Azure DevOps
- Continuous Deployment -> Helm & ArgoCD
- Scripting -> Python
- Monitoring -> Prometheus
- Logging -> Loki
- Observavility Visualization -> Grafana
- Service Mesh -> Istio
- Canary Deployments -> Flagger
- Service Mesh Visualization -> Kiali
  <br/>

<p title="Logos Banner" align="center"> <img  src="https://i.imgur.com/N43tbUi.png"> </p>

<br/>

## Disclaimer

Some things could have been further automated but I prioritized modularization and separation of concerns.<br>

For example, the EKS cluster could have been deployed with ArgoCD installed in one pipeline, but I wanted to have them separated so that each module is focused on its specific task, making each of them more recyclable.

Also, please do submit an issue if you find any errors or you have any good ideas on how to improve this, I would love to hear them.

Let's begin...

<br/>
<br/>
<p title="Automation Everywhere" align="center"> <img width="460" src="https://i.imgur.com/xSmJv0k.png"> </p>
<br/>

---

<br/>

# **LOCAL SETUP**

In order to turn this whole deployment into your own thing, we need to do some initial setup:

1. Fork this repo. Keep the repository name "automate-all-the-things-insane".
1. Clone the repo from your fork:

```bash
git clone https://github.com/<your-github-username>/automate-all-the-things-insane.git
```

2. Move into the directory:

```bash
cd automate-all-the-things-insane
```

2. Run the initial setup script. Come back when you are done:

```bash
python3 python/initial-setup.py
```

4. Hope you enjoyed the welcome script! Now push your customized repo to GitHub:

```bash
git add .
git commit -m "customized repo"
git push
```

5. Awesome! You can now proceed with the Azure DevOps setup.

<br/>
<br/>
<p title="Evil Kermit" align="center"> <img width="650" src="https://i.imgur.com/pIGcI2d.jpg"> </p>
<br/>
<br/>

# **AZURE DEVOPS SETUP**

Before creating our pipelines we need to get a few things set up:<br>

## Create project

1. Sign in [Azure DevOps](https://dev.azure.com/).
2. Go to "New project" on the top-right.
3. Write the name for your project and under "Visibility" select "Private".
4. Click "Create".

<br/>

## Install required plugins

These plugins are required for the pipelines we'll be creating. Click on "Get it free", select your organization and then click "Install".

1. Install [Terraform Tasks plugin](https://marketplace.visualstudio.com/items?itemName=JasonBJohnson.azure-pipelines-tasks-terraform) for Azure Pipelines
1. Install [AWS Toolkit plugin](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-vsts-tools) for Azure Pipelines

<br/>

## Get your AWS keys

These will be required for Azure DevOps to connect to your AWS account.

1. Open the IAM console at https://console.aws.amazon.com/iam/.
2. On the search bar look up "IAM".
3. On the IAM dashboard, select "Users" on the left side menu. _If you are root user and haven't created any users, you'll find the "Create access key" option on IAM > My security credentials. You should know that **creating Access Keys for the root user is a bad security practice**. If you choose to proceed anyway, click on "Create access key" and skip to point 6_.
4. Choose your IAM user name (not the check box).
5. Open the Security credentials tab, and then choose "Create access key".
6. To see the new access key, choose Show. Your credentials resemble the following:

- Access key ID: AKIAIOSFODNN7EXAMPLE<br>
- Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

7. Copy and save these somewhere safe.

<br/>

## Create AWS service connection

This service connection is required for our Azure DevOps pipelines to interact with AWS.

1. Go back to Azure DevOps and open your project.
2. Go to Project settings on the left side menu (bottom-left corner).
3. On the left side menu, under "Pipelines", select "Service connections".
4. Click on "Create service connection".
5. Select AWS, click "Next".
6. Paste your Access Key ID and Secret Access Key.
7. Under "Service connection name", write "aws".
8. Select the "Grant access permission to all pipelines" option.
9. Save.

<br/>

## Create DockerHub service connection

This service connection is required for our Azure DevOps pipelines to be able to push images to your DockerHub registry.

1. While on the "Service connections" screen, click on "New service connection" on the top-right.
2. Select "Docker Registry", click "Next".
3. Under "Registry type" select "Docker Hub".
4. Input your Docker ID and Password.
5. Under "Service connection name", write "dockerhub".
6. Select the "Grant access permission to all pipelines" option.
7. Click on "Verify and save".

<br/>

## Create AWS keys variable group

These are needed for Terraform to be able to deploy our AWS infrastructure.

1. On the left side menu under "Pipelines" go to "Library"
2. Click on "+ Variable group".
3. Under "Variable group name" write "aws-keys".
4. Add the following variables pasting on the "Value" field your AWS keys:

- aws_access_key_id
- aws_secret_access_key

5. Click on the lock icon on each variable.
6. Click on "Save".
7. Click on "Pipeline permissions" and give it "Open access". This means all our pipelines will be able to use these variables.
<p title="Guide" align="center"> <img width="700" src="https://i.imgur.com/aMzTx49.jpg"> </p>

<br/>

## (Optional) Create an Azure self-hosted agent

**If you have a hosted parallelism, you can skip this step.**<br/>

A hosted parallelism basically means that Azure will spin up a server in which to run your pipelines. You can purchase one or you can request a free parallelism by filling out [this form](https://aka.ms/azpipelines-parallelism-request).

If you don't have a hosted parallelism, you will have to run the pipeline in a **self-hosted agent**.
This means you'll install an Azure DevOps Agent on your local machine, which will receive and execute the pipeline jobs.

To install a self-hosted agent on your machine, you can follow the official documentation [here](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#install).

<br/>
<br/>
<p title="We Are Not The Same" align="center"> <img width="460" src="https://i.imgur.com/E0s8TW6.png"> </p>
<br/>
<br/>

---

<br/>
<br/>

# AWS INFRASTRUCTURE DEPLOYMENT PIPELINE

## Description

Our first pipeline, the one that will provide us with all the necessary infrastructure.

What does this pipeline do? If you take a look at the [00-deploy-infra.yml](azure-devops/00-deploy-infra.yml) file, you'll see that the first thing we do is use the Terraform plugin we previously installed to deploy a S3 Bucket and DynamoDB table. These two resources will allow us to store our terraform state remotely and give it locking functionality.<br/>

Why do we need to store our tf state remotely and locking it? Well, this is probably not necessary for this exercise but it's a best practice when working on a team.<br>
Storing it remotely means that everyone on the team can access and work with the same state file, and locking it means that only one person can access it at a time, this prevents state conflicts.

Before we proceed with deploying our actual infrastructure, the pipeline will move the state file to the [terraform/aws/ directory](/terraform/aws/), so our backend resources (the Bucket and DynamoDB Table) will also be tracked as part of our whole infrastructure. If you want to understand how this works, I suggest you watch [this video](https://youtu.be/7xngnjfIlK4?t=2483) where Sid from [DevOps Directive](https://www.youtube.com/@DevOpsDirective) explains it better than I ever could.

Now that the backend is set, we will deploy our actual infrastructure!

So, what is our infra? Well, the main parts are the networking resources, the ElastiCache databases, the EC2 instance and the EKS cluster, along with the Cluster Autoscaler, EBS CSI driver and an AWS Load Balancer Controller which will act as our Kubernetes Ingress Controller.

Having this AWS Load Balancer Controller means that for every Ingress resource we create in our cluster, an AWS Application Load Balancer will be automatically created. This is the native way to do it in EKS and it has a lot to benefits, but it creates an issue for us.<br>
We want to track everything in our infra as IaC, but these automatically created Application Load Balancers won't be tracked in our Terraform... No worries, we'll take care of this issue in the Destroy All The Things Pipeline.<br>
For more info on the AWS Load Balancer Controller you can watch [this excellent video](https://youtu.be/ZfjpWOC5eoE) by [Anton Putra](https://www.youtube.com/@AntonPutra).

In this Insane Edition, apart from the AWS Load Balancer, we'll also be using an Istio Ingress Gateway. Our app (on the three environments) will be accesed through the Istio Ingress Gateway. All other services (ArgoCD, Grafana, etc.) will still be accessed in the previous manner through Ingresses and the AWS Load Balancer. 

If you want to know exactly what is being deployed, you can check out the [terraform files](/terraform/aws). Here you can modify the resources to be deployed to AWS. Let's say you want to add a second EC2 Instance, you can add the following block in the ec2.tf file:

```terraform
resource "aws_instance" "ec2_instance" {
    ami = "ami-01107263728f3bef4"
    subnet_id = aws_subnet.public-subnet-a.id
    instance_type = "t2.micro"
}
```

Commit the changes and run the pipeline again. The backend deployment step will fail, so the pipeline will finish with a warning, you can ignore it.

The pipeline will also modify the [/helm/my-app/backend/environments](helm/my-app/backend/environments) files on the repo. It will get the endpoints for each ElastiCache DB from terraform outputs and include them in the values of each environment.

Oh and lastly... it will export an artifact with the instructions on how to connect to the EC2 instance.

<br/>

## Instructions

1. On your Azure DevOps project, go to "Pipelines" on the left side menu.
2. Select "Pipelines" under "Pipelines" on the left side menu.
3. Click on "Create Pipeline".
4. Select "Github".
5. You might get a screen to authorize Azure Pipelines to access your GitHub account, if so, go ahead and click the green button.
6. Select the repo, it should be "your-github-username/automate-all-the-things-insane"
7. You might also get a screen to install the Azure Pipelines App on your GitHub account, if so, go ahead and click the green button and follow the instructions.
8. Select "Existing Azure Pipelines YAML file".
9. Under "Branch" select "main" and under "Path" select "/azure-devops/00-deploy-infra.yml". Click "Continue".
10. _If you have hosted parallelism skip to point 11_. **If you DON'T have a hosted parallelism**, you need to tell Azure DevOps to use your [**self-hosted agent**](#optional-create-an-azure-self-hosted-agent). In order to do this, you'll need to go to the repo and modify the [00-deploy-infra.yml file](azure-devops/00-deploy-infra.yml).<br>
    Under "pool" you need to edit it so that it looks like this:

```yaml
pool:
  # vmImage: 'ubuntu-latest'
  name: <agent-pool-name> # Insert here the name of the agent pool you created
  demands:
    - agent.name -equals <agent-name> # Insert here the name of the agent you created
```

11. Click on "Run".
12. When it's done, the EC2 instance public IP address will be exported as an artifact. You'll find it in the pipeline run screen. Download it to see the instructions to access the instance.
<p title="Guide" align="center"> <img width="700" src="https://i.imgur.com/UtZyCCe.png"> </p>

<br/>
<br/>
<p title="Pleasure" align="center"> <img width="460" src="https://i.imgur.com/n5lNz3z.jpg"> </p>
<br/>
<br/>

# BACKEND SERVICE BUILD & DEPLOY PIPELINE

## Description

Yes! You're right, we flipped the order of our pipelines in this Insane Edition. This is because we are now using Istio and Flagger. We need the backend and frontend images to be available in the DockerHub registry before deploying their applications. If they don't exist, the canary resources will not deploy properly. Don't worry about it, just carry on with the guide...

Our app is made of two microservices (backend and frontend) and a database. Let's start with the backend.

The [/my-app/backend directory](my-app/backend) on the repo is meant to represent the backend microservice application code repository. Here you'll find the code files and the corresponding Dockerfile for the backend service.

There's four stages on this pipeline:

On the Build stage we will use Docker to build a container image from the Dockerfile, tag it with the number of the pipeline run and push it to your DockerHub registry.

On the Deploy Dev stage, we will checkout the repo and modify the [helm/my-app/backend/environments/values-dev.yaml file](helm/my-app/backend/environments/values-dev.yaml) and push the change to GitHub. [But why?](https://i.gifer.com/2Gg.gif)<br>
This is how we tell ArgoCD that a new version of the backend microservice is available and should be deployed. We modify the image.tag value in the values-dev.yaml file and wait for ArgoCD to apply the changes.

[This is how gentlemen manage their K8S resources](https://i.imgur.com/2Xntz2P.jpg). We are not some cavemen creating and deleting stuff manually with kubectl. We manage our infrastructure with **GitOps**.

After the Deploy Dev stage is done, and only if it was successful, the Deploy Stage stage will commence. It will do the same thing as the previous stage, but this time modifying the [helm/my-app/backend/environments/values-stage.yaml file](helm/my-app/backend/environments/values-stage.yaml).

We'll repeat the same process for Prod, but since Prod should be a more delicate environment, the Deploy Prod stage will require authorization from the top level excecutives (in this case it's you, congrats boss) to be executed. You'll receive an email with a link asking you to verify and approve the deployment to Prod. Go ahead and approve it.

This pipeline is automatically triggered everytime there are any changes commited inside the "backend service application code repository" (meaning the [/my-app/backend directory](my-app/backend)). In this manner, if the backend developers commit any changes to the backend service, they will be automatically built and deployed to the cluster. That's some delicious CI/CD for you baby.

Now, if the infrastrucure team needs to make changes to the cluster resources, they would work on the "K8S infrastructure repository" (meaning the [/helm/my-app directory](helm/my-app)). Let's say they need to increase the number of pod replicas for the backend service in the prod environment, then they'd change the value of deployment.replicas in the [helm/my-app/backend/environments/values-prod.yaml file](helm/my-app/backend/environments/values-prod.yaml), commit the change and wait for ArgoCD to apply the changes on the cluster. There's some tasty Gitops for you too.

<br/>

## Instructions

1. Go to "Pipelines" under "Pipelines" on the left side menu.
2. Click on "New pipeline".
3. Select "GitHub".
4. Select the repo, it should be "your-github-username/automate-all-the-things-insane"
5. Select "Existing Azure Pipelines YAML file".
6. Under "Branch" select "main" and under "Path" select "/azure-devops/01-build-and-deploy-backend.yml". Click "Continue".
7. If you DON'T have a hosted parallelism, you'll need to do the same thing as in point 10 from the [infrastructure deployment pipeline](#instructions).
8. Click on "Run".

<br/>
<br/>
<p title="Momoa & Cavill" align="center"> <img width="460" src="https://i.imgur.com/pCjM1d6.jpg"> </p>
<br/>
<br/>

# FRONTEND SERVICE BUILD & DEPLOY PIPELINE

## Description

The [/my-app/frontend directory](my-app) on the repo is meant to represent the frontend microservice code repository. Here you'll find the code files and the corresponding Dockerfile for the frontend service.

Just as in the backend pipeline, there's four stages on this pipeline:

1. We build and push our frontend image to DockerHub.
2. We deploy to Dev environment in the same manner we did with the backend (modifying the image.tag value in the [helm/my-app/frontend/environments/values-dev.yaml file](helm/my-app/frontend/environments/values-dev.yaml)).
3. If deployment to Dev was OK, we deploy to Stage.
4. If deployment to Stage was OK, we'll get an email requesting authorization for deployment to Prod. Give it the thumbs up and our app will be deployed to Prod.

Just as with the backend, this pipeline is automatically triggered everytime there are any changes commited inside the "frontend service application code repository" (meaning the [/my-app/frontend directory](my-app/frontend)). In this manner, if the frontend developers commit any changes to the frontend service, they will be automatically built and deployed to the cluster.

For the infrastructure, same as before. If the infrastrucure team needs to, for example, change the number of pod replicas for the frontend service in the dev environment, then they'd change the value of deployment.replicas in the [helm/my-app/frontend/environments/values-dev.yaml file](helm/my-app/frontend/environments/values-dev.yaml), commit the change and wait for ArgoCD to apply the changes on the cluster.

<br/>

## Instructions

1. Go to "Pipelines" under "Pipelines" on the left side menu.
2. Click on "New pipeline".
3. Select "GitHub".
4. Select the repo, it should be "your-github-username/automate-all-the-things-insane"
5. Select "Existing Azure Pipelines YAML file".
6. Under "Branch" select "main" and under "Path" select "/azure-devops/02-build-and-deploy-frontend.yml". Click "Continue".
7. If you DON'T have a hosted parallelism, you'll need to do the same thing as in point 10 from the [infrastructure deployment pipeline](#instructions).
8. Click on "Run".

<!-- 9. When it's done, you should be able to access the URLs you got from the ArgoCD Deployment pipeline. But if you go to the URLs too quickly you'll see nothing there. We need to give ArgoCD a little time to notice the changes in the [/helm/my-app/frontend directory](helm/my-app/frontend). By default ArgoCD pulls for changes every three minutes. You can either wait like an adult or go into the ArgoCD web UI and hit "Refresh Apps" like the impatient child that you are.
11. Check the URLs again.
12. On the top left of the website you'll see the "Visit count". This number is being stored in the ElatiCache DB and accessed through the backend.
13. Hope you like the web I made for you. If you did, go leave a star on [my repo](https://github.com/tferrari92/automate-all-the-things-insane). -->

<br/>
<br/>
<p title="Anakin" align="center"> <img width="460" src="https://i.imgur.com/V1qgXKM.jpg"> </p>
<br/>
<br/>

# ARGOCD DEPLOYMENT PIPELINE

## Description

We won't go into what ArgoCD is, for that you have [this video](https://youtu.be/MeU5_k9ssrs) by the #1 DevOps youtuber, Nana from [TechWorld with Nana](https://www.youtube.com/@TechWorldwithNana).

This pipeline will use the [ArgoCD Helm Chart](helm/argo-cd/) in our repo to deploy ArgoCD into our EKS.<br>
The first thing it will do is run the necessary tasks to connect to our the cluster. After this, ArgoCD will be installed, along with its Ingress.

It will then create the necessary resources for ArgoCD to be self-managed and to apply the [App of Apps pattern](https://youtu.be/2pvGL0zqf9o). ArgoCD will be watching the helm charts in the [helm](helm) directory in our repo, it will automatically create all the resources it finds and apply any future changes me make there. The [helm/infra](helm/my-app) and [helm/my-app](helm/my-app) directories simulates what would be our K8S infrastructure repositories would be.

If you want to know more about Helm, [here's another Nana video](https://youtu.be/-ykwb1d0DXU).

Following up, it will get the Istio Gateway endpoint and put it into the [frontend values file](helm/my-app/frontend/values.yaml). It will also export the endpoint for each environment as an artifact.

Finally the pipeline will get the ArgoCD web UI URL and admin account password and export them as an artifact too. You might need to wait a few seconds for the URL to be active, this is because an AWS Load Balancer takes a little time to be deployed.

<br/>

## Instructions

1. Go to "Pipelines" under "Pipelines" on the left side menu.
2. Click on "New pipeline".
3. Select "GitHub".
4. Select the repo, it should be "your-github-username/automate-all-the-things-insane"
5. Select "Existing Azure Pipelines YAML file".
6. Under "Branch" select "main" and under "Path" select "/azure-devops/03-deploy-argocd.yml". Click "Continue".
7. If you DON'T have a hosted parallelism, you'll need to do the same thing as in point 10 from the [infrastructure deployment pipeline](#instructions).
8. Click on "Run".
9. When it's done, the endpoints and ArgoCD access files will be exported as artifacts. You'll find them in the pipeline run screen. Download them to see the ArgoCD URL and credentials, and the frontend endpoints.
<p title="Guide" align="center"> <img width="700" src="https://i.imgur.com/UtZyCCe.png"> </p>

10. You can now access the ArgoCD UI, if it's not ready just hit refresh every few seconds. Here you should find all the applications. Some may still be progressing, be patient.<br>
Three applications will be under the "argocd" project, these are necessary for ArgoCD self-management.<br>
Another three will be under the "observability" project, these are Prometheus, Loki and Grafana.<br>
You will also see a few other applications related to our new service mesh implementation, they will be under the "service-mesh" project. We'll explore these later.<br>
Another six applications will be under the "my-app" project. These manage our app's backend and frontend in the three environments. From the other exported artifact, you'll get the URL for each enviroment's frontend.  

<br/>
<br/>
<p title="Gitops Chills" align="center"> <img width="460" src="https://i.imgur.com/kGQUUTw.jpg"> </p>
<br/>
<br/>

# KUBERNETES TOOLS MANAGEMENT

## Description

Let's talk how we're meant to manage the installation, customization and uninstallation of Kubernetes tools from now on.

In the previos version [Hardcore Edition](https://github.com/tferrari92/automate-all-the-things-hardcore) of this guide, I had you deploy the observability tools through an "Observability Deployment pipeline". What this pipeline did, we'll be doing manually from now on.

If you haven't figured it out yet, let me explain how the system works:<br>
ArgoCD has an Application running which watches the [argo-cd/applications directory](argo-cd/applications). It will deploy all application.yaml's it finds there. Each of these application.yaml's point to their corresponding Helm chart in the [helm directory](helm). This is know as the App of Apps pattern.<br>
When we want to add a new Kubernetes tool to our cluster (let's use Jenkins as an example), we'll do the following:

<br/>

## Instructions

1. Download the Helm chart: after you added the repo, use this command:
```bash
helm pull jenkinsci/jenkins --untar
```
2. Copy the chart to the [helm/infra directory](helm/infra).
3. Create a values-custom.yaml where we'll specify our custom values. We NEVER touch the original values file, we want to have a clear distinction between default configuration and custom configuration.
4. If we need to add a new manifest, we'll create a directory called custom-templates inside the templates directory in the chart and drop our custom manifest in there. 
5. Our chart is ready. We'll now create an application.yaml for it.
6. Copy any of the existing application.yamls and make the required changes. These changes will be on metadata.name, spec.source.path and, depending on what you are deploying, also on spec.destination.namespace and spec.project.
7. Save this new application.yaml in the [argo-cd/applications/infra directory](argo-cd/applications/infra).

That's it! Now you just need to wait. When Argo sees the new application.yaml it will deploy it automatically.<br>
If you need to make any further customizations to the chart, you can modify the values-custom.yaml or the contents of the custom-templates directory.<br>
If you want to remove the tool from your cluster, just delete the application.yaml you created and wait.

We can follow this same logic for deploying new my-app services, for example for a second backend.

<br/>
<br/>
<p title="Two buttons" align="center"> <img width="460" src="https://i.imgur.com/Fgo7nnZ.jpg"> </p>
<br/>
<br/>

# A LITTLE ABOUT ISTIO

As you may have noticed, the [architecture diagram](#what-well-be-doing) has been updated since the [Hardore Edition](https://github.com/tferrari92/automate-all-the-things-hardcore). That's because we are using Istio Service Mesh now.

But what does this mean? Well, as I've said in the past, I'm not going to explain in full detail what Istio or service mesh is, for that you can watch [this Nana video](https://www.youtube.com/watch?v=16fgzklcF7Y&ab_channel=TechWorldwithNana).

What you should know is that Istio will now handle communications inside our cluster. This, as Nana explains, brings a lot of benefits, like advanced traffic management capabilities, improved security and networking observability features, amongst others.

Ingress communications to our cluster will now be handled by the Istio Ingress Gateway instead of a regular Ingress controller as we did before.

We'll also have Kiali, Istio's native visualization tool to see how our communications are handled in a beautiful live graph. You can access by clicking on the little arrow icon the kiali-operator application in ArgoCD's web UI.

But most importantly, Istio will allow us to implement canary deployments. If you don't know what a canary deployment is, [watch this](https://www.youtube.com/watch?v=AWVTKBUnoIg&ab_channel=ByteByteGo).

Istio uses two CRDs (Custom Resource Definitions) to allow us to do this: Virtual Services and Destination Rules. But if you have taken a look into the [my-app helm charts](helm/my-app/), you may have noticed that not only we don't have a service.yaml for our frontend or backend anymore, there's also no virtual-service.yaml nor destination-rule.yaml. "So how does this work?" you may be asking. Well fear not my friend, that's where Flagger comes in...

<br/>

## And what the hell is Flagger?

If you've done some research into canary deployments with Istio, you'll know that that manually orchestrating the canary is a pain in the ass... But do you remember what we are all about here? Exactly, automation.

These missing resources YAMLs we were talking about: Services, VirtualServices and DestinationRules, they will be automatically created and managed by Flagger.

There's a mystery "canary.yaml" manifest in our [my-app helm charts](helm/my-app/). This is a Flagger CRD. It allows us to define how we want our canary deployments to go down. Flagger will read this manifest and take care of creating the necessary resources and operating them according to what we have defined in this canary.yaml. Let's make a little drawing:

<p title="Flagger diagram" align="center"> <img width="800" src="https://i.imgur.com/HLQ3t5l.jpg"> </p>

When we deploy a new version of either our frontend or backend, it won't be instantly deployed to our cluster anymore. Flagger will detect the new version and initialize a canary deployment, moving traffic to our new version gradually and making decisions on if incrementing traffic to the new version or stalling it based on specific metrics, eventually ending on a successful deployment of the new version or on a rollback to the previous one.

The "Primary" resources are the ones that are currently being used and the "Canary" ones are the ones that will be used by the potential new version.

If the canary is successful, the new version will be migrated from "Canary" to "Primary". In the diagram there's this third service (the one that's not Primary nor Canary) that I'm not sure what purpose it serves. I suspect that it is there so that in case of a successful canary deployment, it will ensure there is no downtime when migrating the resources from Canary to Primary, but I haven't found a straight answer tbh.

You can experience the magic of the canary deployment by, for example, changing the background color of the frontend. Once the canary is running you can hit refresh on your browser often and you'll see how it sometimes sends you to the old version (with the old background color) and sometimes to the new (with the new color), gradually sending you more often to the new version until the old one is gone.<br>
You can also see the specific percentage if you click on the Virtual Service object of the frontend application inside ArgoCD's web UI. Just open it and watch how the spec.http.route.weight's in the live manifest change overtime.  

<br/>

## Other important details

We have not completely ditched our previous way of accessing the cluster through regular Ingress objects. We're still using it for accessing the web UIs of our infra tools like ArgoCD, Grafana and Kiali. We switched to Istio Ingress Gateway only for the my-app services.

Also, in [Hardore Edition](https://github.com/tferrari92/automate-all-the-things-hardcore), we had to add TLS encryption to the request from the backend to the ElastiCache DB. This was because we added password protection to the DB, and AWS forces you to use TLS encryption in transit for Elasticache if you want to password protect it. In this case we had to move this TLS origination to the service mesh and remove it form the backend code. Having it in the NodeJS code created a conflict in the communication between the backend container and the envoy container, so we moved it to the service mesh throught the use of a ServiceEntry and a DestinationRule.

Finally, there's another [helm chart for a flagger-loadtester](helm/infra/service-mesh/flagger-loadtester). This is a Flagger tool that will generate fake traffic to our services during the canary deployments. Without this we would have to manually create requests to our services during the canary for the canary to work properly. Flagger will read metrics generated by these requests to make decisions on how to proceed with the canary. Without requests there's no metrics, and without metrics Flagger cannot make decisions, effectively rendering the canary pointless.

<br/>
<br/>
<p title="K8S Architecture" align="center"> <img width="460" src="https://i.imgur.com/weNsDia.jpg"> </p>
<br/>
<br/>

# DESTROY ALL THE THINGS PIPELINE

## Description

Let's burn it all to the ground.

Remember how the AWS Load Balancer Controller created this problem for us where some Applications Load Balancers were created automatically in AWS but were not tracked by our Terraform? Well, in this pipeline, the first thing we need to do it take care of this.

The pipeline will first eliminate ArgoCD from our cluster and then delete all Ingress  and Persistent Volume resources. This will automatically get rid of any Application Load Balancers and EBS volumes in AWS.

After this, the pipeline will be able to run "terraform destroy" with no issues. Our infra will be obliterated and we won't be giving any more of our precious money to Bezos.

The pipeline will finish with a warning, worry not, this is because the "terraform destroy" command will have also deleted our terraform backend (the Bucket and DyamoDB Table), so Terraform won't be able to push the updated state back there. We can ignore this warning. I wish there was a more elegant way of finishing the project but I couldn't find any so deal with it.

## Instructions

1. Go to "Pipelines" under "Pipelines" on the left side menu.
2. Click on "New pipeline".
3. Select "GitHub".
4. Select the repo, it should be "your-github-username/automate-all-the-things-insane"
5. Select "Existing Azure Pipelines YAML file".
6. Under "Branch" select "main" and under "Path" select "/azure-devops/04-destroy-all-the-things.yml". Click "Continue".
7. If you DON'T have a hosted parallelism, you'll need to do the same thing as in point 10 from the [infrastructure deployment pipeline](#instructions).
8. Click on "Run".
9. There's two AWS resources that for some reason don't get destroyed: a DHCP Option Set and an Auto Scaling Managed Rule. I'm pretty sure these don't generate any expenses but you can go and delete them manually just in case. I'm really sorry about this... I have brought [shame](https://i.imgur.com/PIm1apF.gifv) upon my family...

<br/>
<br/>
<p title="Seagull" align="center"> <img width="460" src="https://i.imgur.com/2Z8qEvC.png"> </p>
<br/>
<br/>

---

<br/>

# CONCLUSION

Our journey comes to an end... Congratulations! You made it!

I hope this proved useful (and fun), that you learned something and that you can take some pieces of this to use in your own projects.

You now possess the power of of CI/CD, GitOps, and Infrastructure as Code. You know what [this means](https://www.youtube.com/watch?v=b23wrRfy7SM), use it carefully.

<br/>
<p title="Thanos" align="center"> <img width="500" src="https://i.imgur.com/dgB9Olt.jpg"> </p>
<br/>
<br/>

Special thanks to all these wonderful YouTube people. This wouldn't have been possible without them:

- Nana Janashia from [Techworld With Nana](https://www.youtube.com/@TechWorldwithNana)
- Viktor Farcic from [DevOps Toolkit](https://www.youtube.com/@DevOpsToolkit)
- Marcer Dempers from [That DevOps Guy](https://www.youtube.com/@MarcelDempers)
- Sid Palas from [DevOps Directive](https://www.youtube.com/@DevOpsDirective)
- Mumshad Mannambeth and his guys from [KodeKloud](https://www.youtube.com/@KodeKloud)
- [Anton Putra](https://www.youtube.com/@AntonPutra)

### Happy automating!

<br/>

## On the next edition

[Automate All The Things Braindamage Edition](https://github.com/tferrari92/automate-all-the-things-braindamage):

- ArgoCD Sync Waves
- Sealed Secrets
- Backend code instrumented with OpenTelemetry
- Traces collection and visualization Jaeger
- Removed the use of Canary deployments in Dev environment
