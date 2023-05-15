# Cluster Basics - Create the cluster and verify

## Create cluster
```bash
eksctl create cluster --name=lab1-eks \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --version="1.25" \
                      --without-nodegroup
```
## Get list of clusters
```bash 
eksctl get cluster
```

## Create key pair
```bash 
aws ec2 create-key-pair --key-name lab1-key >> ~/.ssh/lab1.pem
```

## oidc provider 
```bash 
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster lab1-eks \
    --approve
```

## Create EKS nodegroup in private subnet
```bash
eksctl create nodegroup --cluster=lab1-eks \
                        --region=us-east-1 \
                        --name=lab1-eks-ng-private1 \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=4 \
                        --node-volume-size=20 \
                        --ssh-access \
                        --ssh-public-key=lab1-key \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking
```

## Configure kubeconfig for kubectl
```bash
aws eks --region us-east-1 update-kubeconfig --name lab1-eks
```

## Verify cluster: <br>
- List cluster: <br>
`eksctl get cluster`

- List nodegroups in a cluster: <br>
`eksctl get nodegroup --cluster=lab1-eks`

- List nodes in current kubernetes cluster: <br>
`kubectl get nodes -o wide`

- kubectl context changed to new cluster: <br>
`kubectl config view --minify`

## Verify the roles: <br>
- Describe the instances to obtain the role name: <br>
`aws ec2 describe-instances --region us-east-1`

- Once you get the role name with the parameter `IamInstanceProfile` you can use to list the policies attached to the role: <br>
`aws iam list-role-policies --role-name <role-name>` <br>
This command will return a list of policy names attached to the role

- Get policy details: <br>
`aws iam get-role-policy --role-name <role-name> --policy-name <policy-name>`

## Verify Security Group Associated to Worker Nodes: <br>
- Describe the instances to obtain the role name: <br>
`aws ec2 describe-instances --region us-east-1`

- Get security group details: <br>
`aws ec2 describe-security-groups --group-ids <security-group-id> --region <region-name>`

## Verify CloudFormation Stacks: <br>
- List Cloudformation stacks. This command will return a JSON response with all the CloudFormation stacks in the specified region. <br>
`aws cloudformation describe-stacks --region <region-name>`

- Get details for the specific stack. This will provide information about the selected stack, including its status, parameters, outputs, and more. <br>
`aws cloudformation describe-stacks --stack-name <stack-name> --region <region-name>`

- Get stack events to retrieve the events for the specified stack: <br>
`aws cloudformation describe-stack-events --stack-name <stack-name> --region <region-name>`

## Login to Worker Node using Keypair: <br> 
- For Mac, Linux or Windows 10: <br>
`ssh -i lab-key.pem ec2-user@<Public-IP-of-Worker-Node>`

# ======================================================================================= #
# ======================================================================================= #

- Create IAM Policy for the cluster.
```bash
aws iam create-policy --policy-name cluster-admin-policy --policy-document file://cluster-admin-policy.json
```
- Attach policy to a role
```bash
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/cluster-admin-policy --role-name cluster-admin
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookinfo
  namespace: bookinfo
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::698340235880:role/cluster-admin
```
***Make a note of Policy ARN as we are going to use that in next step when creating IAM Role***