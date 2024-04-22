

# MOODLE™ LMS app deployment on AWS EKS with HELM<br><br>

<br>

<p align="center">
  <a href="https://skillicons.dev">
    <img height="50" width="50" src="https://cdn.simpleicons.org/helm/blue" /><img src="https://skillicons.dev/icons?i=kubernetes,aws,docker"/><img height="50" width="50" src="https://hackmd.io/_uploads/rkF2jkVbA.png"/><img height="50" width="50" src="https://github.com/otam-mato/moodle_lms_helm_kubernetes/assets/113034133/31333121-d019-4a9c-9c47-1d6f0b2af94d"/>
  </a>
</p>

<br>

> [!NOTE]
> The deployment of the **MOODLE™ LMS** application<br>
>
> Moodle is a free and open-source learning management system written in PHP and distributed under the GNU General Public License. Moodle is used for blended learning, distance education, flipped classroom and other online learning projects
> 
> Moodle has been developed on the open-source LAMP framework, which consists of **Linux** (operating system), **Apache** (web server), **MySQL** (database), and **PHP** (programming language). While Moodle runs on other technology stacks, we will focus on **LAMP** since it has proven the most popular setup among Moodle administrators. [this link](). 
>
> 
> In the current installment, we're deploying the app using **BITNAMI LMS POWERED BY MOODLE™ LMS** **HELM** CHARTS  on **AWS EKS**.

<br>




## Deployment Strategy

We will be using this Bitnami LMS powered by Moodle™ LMS [HELM charts]() to bootstrap:
   - A Moodle™ deployment on a Kubernetes cluster using the Helm package manager.
   
   - The Bitnami MariaDB chart which is required for bootstrapping a MariaDB deployment for the database requirements of the Moodle™ application.

This approach is the easiest way to get started with our application on Kubernetes. Bitnami application containers are designed to work well together, are extensively documented, and like our other application formats, are continuously updated when new versions are made available.

<br>

## Technologies used
- **Amazon Web Services**
- **AWS EKS**
- **Docker**
- **Kubernetes**
- **HELM**
<br>

## Functionality

Moodle distinguishes between code (primarily written in PHP, HTML, and CSS) and data (mostly values and files added via the Moodle interface).

Moodle libraries, modules (such as resources and activities), blocks, plugins, admin tools, and other entities are represented in code. It is always stored in the filesystem in a Moodle directory referred to as dirroot, which was specified during the installation process in the previous chapter. The code includes all the elements that deal with the backend (server) and frontend (user interface) operations.

Moodle courses, users, roles, groups, competencies, learning plans, grades, and other data, such as learning resources added by educators, forum posts added by learners, and system settings added by the administrator, are mostly stored in the Moodle database. However, files such as user pictures or uploaded assignments are stored in another Moodle directory, known as moodledata, located in a directory called dataroot. Information about files (metadata such as the name, location, last modification, license, and size) is stored in the database, referencing the respective files.

<p align="center">
  <img width="300" alt="Screenshot 2024-01-16 at 20 44 03" src="https://hackmd.io/_uploads/ryTcdJEW0.png">
  <img width="300" alt="Screenshot 2024-01-16 at 20 44 26" src="https://hackmd.io/_uploads/S1TfKyNb0.png">
  <img width="300" alt="Screenshot 2024-01-17 at 16 44 07" src="https://hackmd.io/_uploads/rkGtIlVZR.png">
</p>

<br>

[**Detailed app description**](https://subscription.packtpub.com/book/web-development/9781801816724/2/ch02lvl1sec08/understanding-the-moodle-architecture)

<br>

## Architecture

<p align="center">
  <img width="1400" alt="Screenshot 2024-01-17 at 14 39 48" src="https://hackmd.io/_uploads/ryTcdJEW0.png")
">
  <img width="800" alt="Screenshot 2024-01-16 at 20 43 21" src="https://hackmd.io/_uploads/S1TfKyNb0.png">
</p>

<br>

<br>

## Prerequisites

- A work station or an **EC2 instance**.
- **AWS EKS** cluster [installed](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
- **HELM** [installed](https://helm.sh/docs/intro/install/)
- **Kubectl** [installed](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- **AWS CLI** [installed](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)
- **AWS configured** with a command `aws configure`
- **Kubernetes cluster configured** ([update kubeconfig](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html))

  ```
  aws eks --region eu-west-2 update-kubeconfig --name my-clister
  ```
  
  <img width="800" alt="Screenshot 2024-01-14 at 21 09 50" src="">


<br>

## Steps

### 1. Update the IAM role associated with your EKS worker nodes to include permissions for managing EBS volumes: 

1. Find the IAM role. It should look something like: 
**eksctl-<cluster_name>-nodegroup-ng-NodeInstanceRole-<some_random_number>**

1. Make sure the IAM policy granting EBS volume management permissions **(ec2:CreateVolume)** is attached to the **eksctl-<cluster_name>-nodegroup-ng-NodeInstanceRole-<some_random_number>**.

2. Or, you can create the new policy and attach it to the **eksctl-<cluster_name>-nodegroup-ng-NodeInstanceRole-<somerandom_number>**

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:AttachVolume",
        "ec2:CreateVolume",
        "ec2:DeleteVolume",
        "ec2:DetachVolume",
        "ec2:DescribeVolumes",
        "ec2:DescribeVolumeStatus",
        "ec2:DescribeVolumeAttribute",
        "ec2:DescribeVolumeAttachments"
      ],
      "Resource": "*"
    }
  ]
}

```

### 2. Install the AWS EBS CSI driver:
                     
You will need to deploy the **AWS EBS CSI** driver to your Kubernetes cluster. This driver allows Kubernetes to provision, attach, and mount **EBS volumes** as **persistent volumes**.                     

1. Add the EKS chart repository

```
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver/
```

2. Update your local Helm chart repository cache

```
helm repo update
```

3. Install the AWS EBS CSI driver
```
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver --namespace kube-system --set enableVolumeScheduling=true --set enableVolumeResizing=true --set enableVolumeSnapshot=true
```
                     
### 3. Deploy the **BITNAMI LMS POWERED BY MOODLE™ LMS** **HELM** CHART

1. Deploy the chart:

```
helm install my-release oci://registry-1.docker.io/bitnamicharts/moodle
```

Read more about the installation in the [Bitnami LMS powered by Moodle™ LMS Chart Github repository](https://github.com/bitnami/charts/blob/main/bitnami/moodle/README.md)                     
