##################### Pre-requisite ##########################################################################################
## AWS CLI

## Chocolatey links (installer - for argocd cli deployment)
https://chocolatey.org/install

## Pre-requisite links
https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## kubectl

## Kubectl

## Helm
##############################################################################################################################
##################### Create AWS EKS clsuster ################################################################################
## Create EKS cluster
eksctl create cluster --name eksinsights --node-type t2.large --nodes 1 --nodes-min 1 --nodes-max 2 --region ap-south-1 --zones=ap-south-1a,ap-south-1b,ap-south-1c

## Get EKS Cluster service
eksctl get cluster --name eksinsights --region ap-south-1

## Update Kubeconfig 
aws eks update-kubeconfig --name eksinsights

## Get EKS Pod data.
kubectl get pods --all-namespaces

## Delete EKS cluster
eksctl delete cluster --name eksinsights --region ap-south-1

###################################################### INSTALL WORDPRESS ######################################################
# Create a namespace wordpress
kubectl create namespace wordpress-cwi

# Add the bitnami Helm Charts Repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Deploy WordPress in its own namespace
helm -n wordpress-cwi install understood-zebu bitnami/wordpress

# Check status of deployment
kubectl -n wordpress-cwi rollout status deployment understood-zebu-wordpress

# Get URL to access
kubectl get svc -n wordpress-cwi understood-zebu-wordpress

# /admin login with user name :  user and base 64 decode password from below output
kubectl get secret --namespace wordpress-cwi understood-zebu-wordpress -o jsonpath="{.data.wordpress-password}"

##################################################CREATE AWS EKS Container Insights #######################################################

1.Check if kubectl is working as expected
  kubectl get nodes
  
2.Export the Worker Role Name
  STACK_NAME = eksctl get nodegroup --cluster eksinsights -o json
  aws cloudformation describe-stack-resources --stack-name $STACK_NAME
  
  EX. $ROLENAME : eksctl-eksinsights-nodegroup-ng-0-NodeInstanceRole-13SDOLGQ0C78D
  
3. Add the necessary policy to the IAM role for your worker nodes
  aws iam attach-role-policy --role-name eksctl-eksinsights-nodegroup-ng-0-NodeInstanceRole-13SDOLGQ0C78D --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy
  
4. Verify Role policies if attached.
  aws iam list-attached-role-policies --role-name eksctl-eksinsights-nodegroup-ng-0-NodeInstanceRole-13SDOLGQ0C78D
  
5. kubectl apply -f ./EnableContainerInsights.yaml
   - Create the Namespace amazon-cloudwatch.
   - Create all the necessary security objects for both DaemonSet:
      -SecurityAccount.
      -ClusterRole.
      -ClusterRoleBinding.
   - Deploy Cloudwatch-Agent (responsible for sending the metrics to CloudWatch) as a DaemonSet.
   - Deploy fluentd (responsible for sending the logs to Cloudwatch) as a DaemonSet.
   - Deploy ConfigMap configurations for both DaemonSets.
   
6.  Check the daemonsets
   kubectl -n amazon-cloudwatch get daemonsets
#############################################################################################################################################
