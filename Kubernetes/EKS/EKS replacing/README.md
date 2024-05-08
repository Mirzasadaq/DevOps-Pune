### Install the `AWS CLI` : <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">Docs</a>
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
<img width="708" alt="Screenshot 2022-12-22 at 12 07 08 PM" src="https://user-images.githubusercontent.com/103893307/209091728-43e0232f-fc53-4e32-9b32-e134494fd1b6.png">

### Installing `kubectl` : <a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html">Docs</a>
```
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.7/2022-10-31/bin/linux/amd64/kubectl
```
```
chmod +x ./kubectl
```
```
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```
```
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```
```
kubectl version --short --client
```
<img width="1025" alt="Screenshot 2022-12-22 at 1 50 03 PM" src="https://user-images.githubusercontent.com/103893307/209091884-f6031734-bf69-473f-892f-2a75df9ea57b.png">

### Installing `eksctl` : <a href="https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl">Docs</a>
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```
sudo mv /tmp/eksctl /usr/local/bin
```
```
eksctl version
```
<img width="390" alt="Screenshot 2022-12-22 at 1 52 02 PM" src="https://user-images.githubusercontent.com/103893307/209092048-e91c348d-7a97-47b6-80fe-75a3173ec738.png">

### Configure AWS Command Line using Security Credentials
- Go to AWS Management Console --> Services --> IAM
- Select the IAM User: <your_iam_user_name>
- **Important Note:** Use only IAM user to generate **Security Credentials**. Never ever use Root User. (Highly not recommended)
- Click on **Security credentials** tab
- Click on **Create access key**
- Copy Access ID and Secret access key
- Go to command line and provide the required details

```
aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXX  (Replace your creds when prompted)
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXX  (Replace your creds when prompted)
Default region name [None]: us-east-1
Default output format [None]: json
```
- Test if AWS CLI is working after configuring the above
```
aws ec2 describe-vpcs
```
<img width="779" alt="Screenshot 2022-12-22 at 12 44 15 PM" src="https://user-images.githubusercontent.com/103893307/209093391-412dc3bc-f6f4-40b3-a43e-bd78f926d218.png">
# Create EKS cluster using single CMD

```
eksctl create cluster --name eksdemo --version 1.23 --region us-east-1 --nodegroup-name eksdemo-ng --node-type t3.medium --nodes 2 --managed
```
### Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command.           
#### Template
```
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve
```
#### Replace with region & cluster name

```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo \
    --approve
```

# Install-EBS-CSI-Driver
https://docs.aws.amazon.com/eks/latest/userguide/csi-iam-role.html 

https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html


# EKS Storage -  Storage Classes, Persistent Volume Claims

## Introduction
- We are going to create a MySQL Database with persistence storage using AWS EBS Volumes

| Kubernetes Object  | YAML File |
| ------------- | ------------- |
| Storage Class  | storage-class.yml |
| Persistent Volume Claim | persistent-volume-claim.yml   |
| Config Map  | UserManagement-ConfigMap.yml  |
| Deployment, Environment Variables, Volumes, VolumeMounts  | mysql-deployment.yml  |
| ClusterIP Service  | mysql-clusterip-service.yml  |

## Create following Kubernetes manifests
### Create Storage Class manifest
- https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode
- **Important Note:** `WaitForFirstConsumer` mode will delay the volume binding and provisioning  of a PersistentVolume until a Pod using the PersistentVolumeClaim is created. 

### Create Persistent Volume Claims manifest
```
# Create Storage Class & PVC
kubectl apply -f kube-manifests/

# List Storage Classes
kubectl get sc

# List PVC
kubectl get pvc 

# List PV
kubectl get pv
```
### Create ConfigMap manifest
- We are going to create a `usermgmt` database schema during the mysql pod creation time which we will leverage when we deploy User Management Microservice. 

### Create MySQL Deployment manifest
- Environment Variables
- Volumes
- Volume Mounts

### Create MySQL ClusterIP Service manifest
- At any point of time we are going to have only one mysql pod in this design so `ClusterIP: None` will use the `Pod IP Address` instead of creating or allocating a separate IP for `MySQL Cluster IP service`.   

## Create MySQL Database with all above manifests
```
# Create MySQL Database
kubectl apply -f kube-manifests/

# List Storage Classes
kubectl get sc

# List PVC
kubectl get pvc 

# List PV
kubectl get pv

# List pods
kubectl get pods 

# List pods based on  label name
kubectl get pods -l app=mysql
```

## Connect to MySQL Database
```
# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
```

# MySQL Commands 

Here are some common MySQL database commands with examples:

1. Connect to MySQL server:
```shell
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11
```

2. Create a new database:
```sql
CREATE DATABASE mydatabase;
```
Creates a new database named `mydatabase`.

3. Show existing databases:
```sql
SHOW DATABASES;
```
Lists all the databases available on the MySQL server.

4. Use a specific database:
```sql
USE mydatabase;
```
Switches to the `mydatabase` database for executing subsequent commands.

5. Create a new table:
```sql
CREATE TABLE mytable (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  age INT
);
```
Creates a new table named `mytable` with columns `id`, `name`, and `age`.

6. Insert data into a table:
```sql
INSERT INTO mytable (name, age) VALUES ('John', 25);
```
Inserts a new row with name `John` and age `25` into the `mytable` table.

7. Select data from a table:
```sql
SELECT * FROM mytable;
```
Retrieves all rows and columns from the `mytable` table.

8. Update data in a table:
```sql
UPDATE mytable SET age = 30 WHERE name = 'John';
```
Updates the `age` to `30` for the row with name `John` in the `mytable` table.

9. Delete data from a table:
```sql
DELETE FROM mytable WHERE name = 'John';
```
Deletes the row with name `John` from the `mytable` table.

10. Drop a table:
```sql
DROP TABLE mytable;
```
Removes the `mytable` table from the database.

### Note : These are just a few examples of commonly used MySQL database commands. There are many more commands and features available in MySQL for managing databases, tables, and data.
