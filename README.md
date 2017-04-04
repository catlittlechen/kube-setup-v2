# Setup a secure K8 cluster

---> Decide on a <Cluster Name> which is also its DNS access point like kube8.rockyj.de

1. Setup AWS CLI

2. Download KOPS and put it in PATH

3. Create VPC using my ansible-script (https://github.com/rocky-jaiswal/ansible-aws-vpc)

4. Check the tags of the subnets and routing tables, it should have KubernetesCluster = <Cluster Name> tag. --> __VERY IMPORTANT__

5. Create S3 bucket -
aws s3api create-bucket --bucket k8-state-03april2017-v4 --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1

6. Setup cluster
kops create cluster --zones eu-central-1a --topology=private --networking=calico --master-size=m3.medium --node-size=m3.medium --bastion --state=s3://k8-state-03april2017-v4 --vpc=vpc-b748c9df --network-cidr=10.0.0.0/16 kube10.rockyj.de

7. Edit for VPC
kops edit cluster kube10.rockyj.de --state=s3://k8-state-03april2017-v4

On update delete the prefilled CIDRs, and replace them with the subnet ID, the utility subnet will be the public subnet and rest private

8. Apply cluster
kops update cluster kube10.rockyj.de --state=s3://k8-state-03april2017-v4 --yes

9. Validate cluster
kops validate cluster kube10.rockyj.de --state=s3://k8-state-03april2017-v4

10. get Kube config
mv ~/.kube/config ./kubeconfig

11.
kubectl --kubeconfig=./kubeconfig get nodes


# Install a service and setup ELK

- Apply all K8 services

- Apply the simple web

- Describe service to find ELB URL

kubectl --kubeconfig=./kubeconfig describe service simple-web

- Apply FluentdDS, ES and Kibana service definitions

- See all pods

kubectl --kubeconfig=./kubeconfig get pods --namespace=kube-system

- Delete ES pods, e.g.

kubectl --kubeconfig=./kubeconfig delete pod elasticsearch-logging-v1-13mj5 --namespace=kube-system

Boohoo! We lost all logs!! Unacceptable!
