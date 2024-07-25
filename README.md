
## 1. Skaffold

Skaffold is a command line tool that facilitates continuous development for Kubernetes applications. You can iterate on your application source code locally and then deploy it to local or remote Kubernetes clusters. Skaffold handles the workflow for building, pushing, and deploying your application. It also provides building blocks and describes customizations for a CI/CD pipeline.

### Step1: Installing Skaffold

```
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && \sudo install skaffold /usr/local/bin/
```
Verify the skaffold version

```
skaffold version
```

The skaffold manifest for Dunsample is named as skaffold.yaml in the repository

To display and authenticate the Amazon ECR Registry

```
aws ecr get-login-password --region <region> | docker login --password-stdin --username AWS <account_id>.dkr.ecr.<region_id>.amazonaws.com
```
To authenticate  the container registry which pulls the private image, use the following command.

```
kubectl create secret generic aws-registry --from-file=<path_to_the_config_file> --type=kubernetes.io/dockerconfigjson
```

### Step2: Installing Terraform

Ubuntu/Debian

Download the latest version of Terraform

```
wget https://releases.hashicorp.com/terraform/1.0.7/terraform_1.0.7_linux_amd64.zip
```

Extract the downloaded file archive

```
unzip terraform_1.0.7_linux_amd64.zip
```

Move the executable into a directory searched for the executables

```
sudo mv terraform /usr/local/bin/
```

Check the version of the downloaded Terraform

```
terraform version
```

### Step3: Installing AWS CLI

Update the APT package repository cache with the following command:

```
$ sudo apt-get update
```
Install AWS CLI on Ubuntu 22.04 LTS from the official package repository, run the following command:
```
$ sudo apt-get install awscli -y
```
Test the version of AWS CLI
```
$ aws --version
```

## 3. Creating EKS Cluster using Terraform

Before creating EKS Cluster, you'll need the following:

1. An AWS account

2. Identity and access management (IAM) credentials and programmatic access

3. AWS credentials that are set up locally <aws configure>

OR 

```
aws sso configure
aws sso login --profile <profile_name>
```


  
Now we can continue with our resource creation using Terraform.
We will have terraform files with different resource configurations with the file extension  *.tf

The Terraform directory in GitHub for Dunavant consists of the required resource files which need to be applied in the order of VPC first,  EKS Cluster, and then worker nodes.

Inside the **vpc/eks-vpc** folder, run the following commands.

```
terraform init  #initialize a working directory containing Terraform configuration files
terraform plan  #creates an execution plan, which lets you preview the changes that Terraform plans to make to your infrastructure
terraform apply  #carries out the planned changes
```
The same instruction is to be followed for **eks-worker-node** first and then **eks-cluster** 

Redis, Postgres and Mailhog are required as application is dependent on them. In order to install these, go to Helmchart directory in the repo and run the following commands,

```
helm install mailhog mailhogsample/
helm install redis redissample/
helm install postgresql postgresqlsample/
```
  
Skaffold  can sync the changes in the local kind cluster. Before making commits in the github branch, test it locally.

```
git add .
git commit -m "message"
```

Run the Skaffold command to see the changes locally before pushing in the repo.
```
skaffold run
```
If everything is good and working as expected, developer can push the branch to remote.

```
git push
```

The changes are triggered through the CI/CD pipeline created using Github action as soon as the changes are pushed in the repository.
The workflow is located in:
```
.github/workflows/main.yaml 
```
  
Commands to access the endpoints of rose, mailhogsample and django application
  
```
  kubectl get service   #to get all the services
```
  
  Access the django application
  
  ```
  kubectl port-forward svc/dunsample 8000   #to access the django application
  ```
Access the rose endpoint
  
```  
  kubectl port-forward svc/rose 5555 
```

  Access the mailhogsample endpoint

```
  kubectl port-forward svc/mailhogsample 8025    
```
