## Project Overview
*The setup automates the deployment of a SonarQube server, Nexus Server and Kubeadm master and worker node, using Terraform Infrastructure as Code (IaC).*

#### Main components:

- ➡️ Custom **VPC** with public subnets, Internet Gateway, and route tables
- ➡️ **Security Groups** for Sonar, Nexus and kubernetes cluster
- ➡️ EC2 Instance for SonasrQube (with User Data installation script)
- ➡️ EC2 Instance for Nexus (with User Data installation script)
- ➡️ EC2 Instances for Kubeadm (with User Data installation script)

## Prerequisites
Before Running Terraform, Make sure you have the following prerequisites ready:

- ➡️ Terraform v1.3+ (recommended)
- ➡️ AWS CLI configured with proper IAM credentials
- ➡️ Public and Private Key
