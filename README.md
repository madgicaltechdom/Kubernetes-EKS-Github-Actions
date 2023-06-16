# Kubernetes-EKS-Github-Actions

The main goal of this project is:
  - Create a website with a node
  - The site uses unit tests to validate its proper functioning
  - Create an ECR registry
  - Create a Kubernetes cluster on EKS with eksctl
  - Put the application online with GitHub Actions, kubetctl, and kustomize
  - Each git push should automatically update the Kubernetes cluster if the tests pass

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/5e0be5b8-bd2f-4a07-826b-b4b82250391d)

So, in this project, I will show you the following points:

  1. Install, setup, and explore the project
  2. Creating the cluster and setting up the configuration file
  3. Deployment with GitHub actions
  4. Updating the website
  5. At last, delete our cluster 

# Prerequisites
  - jq and wget must be installed.
  - KUBECTL Version must be "v1.23.6"

# Install, set up, and explore the project
Step-by-step user guide [video](https://drive.google.com/file/d/1W7lyXr4p1Mr3JwXX40kw9NmxCA1vNa69/view?usp=sharing).

1. Run the below command to clone this repo:

```
git clone https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions.git
```
2. Provide the AWS_REGION where you want to create these things in line no. 5 in the make.sh file.

3. Provide the PROJECT_NAME in line no. 7 in the make.sh file.

4. To set up the project, run the following command:

```
# install eksctl + kubectl + yq, create aws user + ecr repository
make setup
```
5. The above command will Install eksctl if it is not already installed, Install kubectl if it is not already installed, Install yq if it is not already installed, Creates an AWS Power User for the project, Creates an ECR repository, and Creates an .env file from the .env.tmpl file.

The user is created:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/30f8b453-bb0d-4c27-a184-64500ae7879f)

The ECR repository is created:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/6ebed24d-ab5d-4bec-8974-dccc472c3398)

6. For testing the site run the below command:

```
# local development (by calling npm script directly)
make dev
```
7. By opening the address http://localhost:3000 you can see the site:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/efb756f0-d44d-462e-8e64-b3f552db2bad)

8. We can stop the website with Ctrl + C.

9. Run the tests with the below command:

```
# Run tests (by calling the npm script directly)
make test
```
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/68cf7e75-15c3-4d8e-b290-693f18e88b95)

# Creating the cluster and setting up the configuration file
Step-by-step user guide [video](https://drive.google.com/file/d/1Z0jYnAbTbeaO02MwGaYszh_SberZnwgu/view?usp=sharing).

1. For creating the EKS cluster run the below command:

```
make cluster-create
```
We launch the creation of the EKS cluster. You have to be patient because it takes about 15 minutes!
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/fc61c383-fbea-4f0f-ac6f-9ad4987594ac)

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

4. Run the below command through this command, the certificate is hidden: DATA+OMITTED. Also, note that this configuration file can contain the accesses of multiple clusters.

```
kubectl config view
```

5. As we do not want to use our root administrator accesses but that of the user we just created, we need to remove AWS_STS_REGIONAL_ENDPOINTS and AWS_PROFILE variables. All of these steps can be done with the following command:

```
# create kubectl EKS configuration
make cluster-create-config
```
This command creates the kubeconfig.yaml configuration file and aws-auth-configmap.yaml file. It also creates a KUBECONFIG file which is the same file, encoded in base64.
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/893c8a9d-ab8f-4e6b-9761-31afde76022d)

6. We now need to modify our cluster so that this new user can administer it. So for seeing the current state of our configuration run the below command:

```
kubectl -n kube-system get configmap aws-auth -o yaml
```
7. Copy the "mapUsers" field from the output of 5th step command and past it in aws-auth-configmap.yaml file same as shown below:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/04cef667-b75b-41f4-a530-5ccbeaf599e4)

8. Apply the modified file to the cluster by running this command:

```
# apply kubectl EKS configuration
make cluster-apply-config
```
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/455aad0f-b9a2-4102-ad2e-67ceac82ccc3)

# Deployment with GitHub actions

The video is given in the reference.

1. Make the changes in the .github/workflows/cd.yml file like AWS_REGION and ECR_REPOSITORY.

2. Create the below secrets in your repo.

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/07cbfcea-0620-4fed-8fbc-d0ae6befa4c9)

3. To update the cluster, we use the manifests from the k8s directory. In the Deploy step of our GitHub actions file, the following command is used to generate the kustomization.yaml file:

```
export ECR_REPOSITORY="$ECR_REGISTRY/$ECR_REPOSITORY"
export IMAGE_TAG="$SHORT_SHA"
envsubst < k8s/kustomization.tmpl.yaml > k8s/kustomization.yaml
```

4. Then we apply these transforms using the following command:

```
export KUBECONFIG=kubeconfig.yaml
kubectl kustomize k8s | kubectl apply -f -
```

5. We will test our script by committing:

```
git push
```

The action is successful:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/f3448373-62f9-4e41-aea2-ad3a3b6820e9)

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/6c76b6c1-b3dd-4cf1-bdf7-bef357c2e22b)

6. We query our cluster:

```
kubectl get ns
```

7. To get the URL of our Load Balancer we run the following command, If it is not working then you can take it from the AWS account:

```
make cluster-elb
```

By using this URL in our browser we can see our website:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/eac8c415-4f10-4272-b410-1cf375ef7fed)

A docker image has been pushed to our repository:
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/5dec5481-ae01-4ada-aafb-d37dce12bc0b)

8. For testing whether our workflow is working fine or not we will add an error by deactivating a route in the app.js file:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/05893e84-17c0-49a6-b86b-f47669d53596)

9. Then we push the code:

```
git add .
git commit -m :boom:
git push
```

10. The process is interrupted by the error:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/8e32d7ed-a95c-4a70-8d4d-991fdc2cb4d6)

11. We edit the app.js file again to remove the error and again push the code:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/d9196dc1-08f8-4665-85b9-15f933c15555)

12. By reloading our browser, we see that our site is working properly:

![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/a917e760-360e-41f7-b9ca-a59b5e84b6f3)

13. We can delete our cluster with this command:

```
make cluster-delete
```
![image](https://github.com/Shubhammadgical/Kubernetes-EKS-Github-Actions/assets/101810595/493ba199-7b88-48ec-bb02-fb261f6b3c2e)

# Reference

I got information from this [article](https://medium.com/@jerome.decoster/kubernetes-eks-github-actions-a874321fb9b4). This is the step-by-step user guide [video](https://drive.google.com/file/d/1FtUGLTKm_EMbYgahOgbjN6fQiLmq-fNR/view?usp=sharing).
