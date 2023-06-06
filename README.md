# Kubernetes-EKS-Github-Actions

The main goal of this project is:
  - Create a website with node
  - The site uses unit tests to validate its proper functioning
  - Create an ECR registry
  - Create a Kubernetes cluster on EKS with eksctl
  - Put the application online with Github Actions, kubetctl and kustomize
  - Each git push should automatically update the Kubernetes cluster if the tests pass

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/5e0be5b8-bd2f-4a07-826b-b4b82250391d)

So, in this project I will show you the follow points:

  1. Install, setup, and explore the project
  2. Creating the cluster and setting up the configuration file
  3. Deployment with GitHub actions
  4. Updating the website
  5. At last, delete our cluster 

# Prerequisites
  - jq and wget must be installed.

# Install, setup and explore the project

1. Run the below command to clone this repo:

```
git clone https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions.git
```
2. Provide the AWS_REGION where you want to create these things in line no. 5 in the make.sh file.

3. Provide the PROJECT_NAME in line no. 7 in the make.sh file.

4. To setup the project, run the following command:

```
# install eksctl + kubectl + yq, create aws user + ecr repository
$ make setup
```
5. The above command will Install eksctl if it is not already installed, Install kubectl if it is not already installed, Install yq if it is not already installed, Creates an AWS Power User for the project, Creates an ECR repository, and Creates an .env file from the .env.tmpl file.

The user is created:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/30f8b453-bb0d-4c27-a184-64500ae7879f)

The ECR repository is created:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/6ebed24d-ab5d-4bec-8974-dccc472c3398)

6. For testing the site run the below command:

```
# local development (by calling npm script directly)
$ make dev
```
7. By opening the address http://localhost:3000 you can see the site:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/efb756f0-d44d-462e-8e64-b3f552db2bad)

8. We can stop the website with Ctrl + C.

9. Run the tests with the below command:

```
# run tests (by calling npm script directly)
$ make test
```
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/68cf7e75-15c3-4d8e-b290-693f18e88b95)

# Creating the cluster and setting up the configuration file

1. For creating the EKS cluster riun the below command:

```
make cluster-create
```
We launch the creation of the EKS cluster. You have to be patient because it takes about 15 minutes!
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/872db31b-8efe-496a-bd2f-745cd6824fb4)

The cluster is created:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/2028a622-3da3-4f26-968f-77fbd97049be)

2. Once the deployment is complete, we check that kubectl points to the right cluster:

```
kubectl config current-context
```
My output: shubham@kubernetes-github-actions-shubham-test.ap-south-1.eksctl.io

3. The configuration of kubectl is done by a YAML file. On our local machine, the configuration file is located here:

```
cat $HOME/.kube/config
```

4. Run the below command through this command, the certificate is hidden : DATA+OMITTED. Also note that this configuration file can contain the accesses of multiple clusters.

```
kubectl config view
```

5. As we do not want to use our root administrator accesses but that of the user we just created, we need to remove AWS_STS_REGIONAL_ENDPOINTS and AWS_PROFILE variables. All of these steps can be done with the following command:

```
# create kubectl EKS configuration
$ make cluster-create-config
```
This command creates the kubeconfig.yaml configuration file. It also creates a KUBECONFIG file which is the same file, encoded in base64.
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/6615e7fb-c048-4759-b1d7-f3ec472c8d22)

6. We now need to modify our cluster so that this new user can administer it. So for seeing the current state of our configuration run the below command:

```
kubectl -n kube-system get configmap aws-auth -o yaml
```




