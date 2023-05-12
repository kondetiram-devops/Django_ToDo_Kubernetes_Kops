1. Launch a Ubuntu Server for KOPS

2. Install AWSCLI:

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o"awscliv2.zip"

sudo apt install unzip

unzip awscliv2.zip

sudo ./aws/install


3. Install KOPS:

sudo apt install wget -y

sudo curl -LO https://github.com/kubernetes/kops/releases/download/v1.20.0/kops-linux-amd64

sudo chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops


4. Install Kubectl:

sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin

sudo kubectl version


5. Create IAM Role from Console or CLI with below Policies:

AmazonEC2FullAccess

AmazonS3FullAccess

IAMFullAccess

AmazonVPCFullAccess

Then attach the IAM Role to the running KOPS Ubuntu Server from the Intance Actions --> Security --> Modify IAM Role --> Select the IAM Role created

* If you want to use KOPS server local not hosted on AWS then we need to create a User in AWS IAM and attach the same above policies to it 


6. Create a S3 Bucket (To have etcd stored because to have HA) and use unique name:

aws s3 mb s3://<bucket-name>.k8s.local


7. Add and Expose enviroment variables:

vi .bashrc

export NAME=<cluster-name>.k8s.local
export KOPS_STATE_STORE=s3://<bucket-name>.k8s.local

source .bashrc


8. Creating SSH Keys to connect to the cluster nodes

ssh-keygen


9. Create kubernetes cluster definitions on S3 Bucket

kops create cluster \
--state=${KOPS_STATE_STORE} \
--node-count=2 \
--master-size=t3.medium \
--node-size=t3.medium \
--zones=us-east-1a \
--name=${NAME} \
--dns private \
--master-count 1

10. Create a secret for KOPS to copy SSH

kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub


11. Create K8 Cluster:

kops update cluster ${NAME} --yes --admin


12. Validate your Cluster:

kops validate cluster


13. To list nodes:

kubectl get nodes


14. Delete Cluster:

kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes








