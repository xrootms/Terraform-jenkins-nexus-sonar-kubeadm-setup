# Automated Terraform CI/CD Pipeline with Checkov Security Scanning (Jenkins)
This project demonstrates a fully automated **Terraform CI/CD pipeline** built using **Jenkins Declarative Pipeline**, with integrated **security scanning using Checkov**. 
The pipeline provisions infrastructure while enforcing DevSecOps best practices.


## Project Overview
This project focuses on building a fully automated and secure **Infrastructure-as-Code** (IaC) deployment pipeline using Terraform, integrated with Jenkins CI/CD and Checkov security scanning.
The pipeline provisions a complete AWS infrastructure stack that includes:

- ➡️ A custom VPC with public subnets, Internet Gateway, and route tables
- ➡️ Security Groups for SonarQube, Nexus, and Kubernetes cluster nodes
- ➡️ EC2 instances for SonarQube, Nexus Repository Manager, and Kubeadm master/worker nodes (with User Data installation script)

## Prerequisites
Before Running This project, Make sure you have the following prerequisites ready:

### Local / Developer Requirements

➡️ Terraform v1.3+ installed
➡️ AWS CLI configured with IAM user credentials (Access Key & Secret Key)
➡️ Public and Private Key
➡️ Git installed to clone the repository
➡️ Basic IAM permissions for Terraform (EC2, VPC, IAM, S3, CloudWatch)

### Jenkins Requirements

➡️ Jenkins installed on a server (EC2, VM etc.)
➡️ Required Jenkins plugins: AWS Credentials
➡️ Jenkins credentials configured for: Github, AWS kyes and terraform.tfvars file
➡️ Checkov (via pip) Installed 
➡️ Terraform v1.3+ installed

### Terraform Backend (Optional but recommended)

➡️ S3 bucket for remote backend state
➡️ DynamoDB table for state locking


### Create a Jenkins Pipeline Job*

**Path:** `Jenkins > New Item`

1. Name: `full-stack`
2. Type: `Pipeline`
3. Discard Old Builds → Keep for 100 days, max 2 builds
4. Add pipeline script:

```groovy
pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/xrootms/Terraform-Nexus-SonarQube-Kubeadm-setup.git'
            }
        }
        stage('Load tfvars') {
            steps {
                withCredentials([file(credentialsId: 'tvars', variable: 'TFVARS')]) {
                  sh '''
                  ls -l
                  rm -f terraform.tfvars
                  ls -l
                  cp $TFVARS terraform.tfvars
                  ls -l
                  '''
                }
            }
        }
        stage('terraform init') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                  sh 'terraform init'
              }
            }
        }
        stage('Checkov Scan') {
            steps {
                catchError(buildResult: 'SUCCESS') {
                    // Run Checkov and save report directly
                    sh "checkov -d . -o junitxml --output-file-path checkov-report.xml"
                    
                    // Publish Checkov JUnit report in Jenkins
                    junit testResults: 'checkov-report.xml', skipPublishingChecks: true
                 }
                 // Archive the report
                 archiveArtifacts artifacts: 'checkov-report.xml', onlyIfSuccessful: false
             }
          }
       stage('terraform plan') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                  sh 'terraform plan -var-file=terraform.tfvars -out=tfplan'
              }
            }
        }
        /*stage('terraform Apply') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-creds', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                  sh 'terraform apply -auto-approve -var-file=terraform.tfvars'
              }
            }
        }*/
        
    }
}


```



####  Clone the repo:
   ```bash
   git clone https://github.com/xrootms/Terraform-Nexus-SonarQube-Kubeadm-setup.git
   cd Terraform-Nexus-SonarQube-Kubeadm-setup
   ```

#### 2. Copy and edit variables: (Update variable values as needed — VPC, CIDR, public key, region, etc.)
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   ```

#### 3. Initialize Terraform:
   ```bash
   terraform init
   ```

#### 4. Plan and Apply:
   ```bash
   terraform plan
   terraform apply
   ```

#### 5. Get ssh connection for EC2:

