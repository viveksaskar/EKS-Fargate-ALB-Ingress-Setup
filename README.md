AWS Elastic Kubernetes Service (EKS) Project Guide
AWS Elastic Kubernetes Service (EKS) is a managed service that simplifies running Kubernetes on AWS without needing to install and operate your own Kubernetes control plane or nodes. EKS is particularly effective when used with AWS Fargate, which allows you to run Kubernetes pods without managing the underlying infrastructure.

While EKS manages the control plane, it also offers the flexibility to manage worker nodes, attaching them to the master nodes as needed. Alternatively, you could use KOPS for managing Kubernetes, though it requires more hands-on management.

Managing Kubernetes can be complex, and using a load balancer can be costly. It's often better to use an ingress controller, such as the AWS Application Load Balancer (ALB) controller, for more efficient load balancing.

Configuration
A DevOps engineer will need to create the necessary Kubernetes resources, including pods, services, and ingress, for AWS EKS.

Steps to Set Up EKS
Install Required Tools:

kubectl: Kubernetes command-line tool.
eksctl: Command-line tool for EKS.
awsCLI: Command-line tool for AWS.
Create an EKS Cluster:

Use the following command to create a cluster with Fargate support:
sh
Copy code
eksctl create cluster --name <cluster-name> --region <region> --fargate
This process can take some time and will configure the cluster with necessary public and private IPs, and provide a dashboard view.
Configure kubectl for EKS:

Update your kubeconfig to use the new EKS cluster:
sh
Copy code
aws eks update-kubeconfig --name <cluster-name> --region <region>
Create a Fargate Profile:

Add an additional Fargate profile:
sh
Copy code
eksctl create fargateprofile --cluster <cluster-name> --region <region> --name alb-sample-app --namespace game-2048
Deploy Application:

Apply the configuration for your application:
sh
Copy code
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
Verify that the resources are created:
sh
Copy code
kubectl get pods -n game-2048
kubectl get svc -n game-2048
kubectl get ingress -n game-2048
Setting Up the ALB Ingress Controller
Create an IAM OIDC Provider:

Associate the OIDC provider with your cluster:
sh
Copy code
eksctl utils associate-iam-oidc-provider --cluster <cluster-name> --approve
Download and Create IAM Policy:

Download the IAM policy:
sh
Copy code
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
Create the IAM policy:
sh
Copy code
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
Create an IAM Role for the ALB Controller:

Create the IAM role and associate it with your cluster:
sh
Copy code
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
Deploy the ALB Controller Using Helm:

Add and update the Helm repository:
sh
Copy code
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
Install the ALB controller:
sh
Copy code
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
Verify that the ALB controller is running:
sh
Copy code
kubectl get deployment -n kube-system aws-load-balancer-controller
Troubleshooting
If you encounter errors, you can debug the deployment by checking the status of your resources in AWS CloudFormation. Navigate to the CloudFormation console, find the relevant stack, and review or delete it if necessary.

By following these steps, you can efficiently set up and manage your Kubernetes applications on AWS EKS with the ALB ingress controller.
