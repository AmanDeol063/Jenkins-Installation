## Overview

This guide walks you through deploying Jenkins—a leading open-source automation server—on an AWS EC2 instance using the AWS CLI and standard package managers. It covers prerequisites, EC2 provisioning, SSH access, Jenkins installation, initial setup, optional enhancements, and basic troubleshooting.

---

## Prerequisites

1. **AWS Account & CLI**
   You must have an AWS account with permissions to create EC2 instances, security groups, and key pairs, and the AWS CLI installed and configured (`aws configure`). ([AWS Documentation][1])
2. **EC2 Key Pair**
   Prepare an EC2 key pair (e.g., `my-aws-key.pem`) to SSH into the instance. ([AWS Documentation][2])
3. **IAM Role (Optional)**
   For Jenkins jobs accessing AWS services (S3, CodeCommit), attach an IAM role to the instance. ([Jenkins][3])

---

## AWS EC2 Instance Setup

1. **Choose an AMI**

   * Amazon Linux 2 (x86\_64) or Ubuntu 20.04 LTS. ([Jenkins][3])
2. **Launch EC2 via AWS CLI**

   ```bash
   aws ec2 run-instances \
     --image-id ami-0123456789abcdef0 \
     --instance-type t3.medium \
     --key-name my-aws-key \
     --security-group-ids sg-0123456789abcdef0 \
     --subnet-id subnet-0123456789abcdef0 \
     --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=jenkins-server}]'
   ```

   This command launches an EC2 instance using the specified AMI and VPC resources. ([AWS Documentation][1])
3. **Configure Security Group**
   Ensure inbound rules allow SSH (port 22), HTTP (port 80), and Jenkins (port 8080) from appropriate sources. ([GeeksforGeeks][4])

---

## SSH into EC2 and Install Jenkins

1. **Connect via SSH**

   ```bash
   chmod 400 my-aws-key.pem
   ssh -i my-aws-key.pem ec2-user@<EC2_PUBLIC_IP>
   ```

   Replace `<EC2_PUBLIC_IP>` with your instance’s public IP. ([GeeksforGeeks][4])
2. **Update System & Install Java 11**
   On Amazon Linux 2:

   ```bash
   sudo yum update -y
   sudo amazon-linux-extras install java-openjdk11 -y
   ```

   Jenkins requires Java 11 or newer. ([Stack Overflow][5])
3. **Add Jenkins Repo & Install**

   ```bash
   # Import Jenkins GPG key
   sudo wget -O /etc/yum.repos.d/jenkins.repo \
     https://pkg.jenkins.io/redhat-stable/jenkins.repo
   sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

   # Install Jenkins
   sudo yum install jenkins -y
   ```

   On Ubuntu, you would use `apt-key add` and `apt-get install jenkins` instead. ([Jenkins][6], [DigitalOcean][7])
4. **Start & Enable Jenkins**

   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

   This ensures Jenkins launches on boot. ([Jenkins][3])
5. **Verify Status**

   ```bash
   sudo systemctl status jenkins
   ```

   Confirm the service is active and running. ([Jenkins][8])

---

## Initial Jenkins Configuration

1. **Access the Web UI**
   Navigate to `http://<EC2_PUBLIC_IP>:8080` in your browser. ([Jenkins][3])
2. **Retrieve Initial Admin Password**

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

   Copy and paste this into the setup wizard. ([Stack Overflow][9], [GeeksforGeeks][10])
3. **Install Suggested Plugins**
   Choose “Install suggested plugins” to get a typical CI/CD toolchain. ([Jenkins][3])
4. **Create First Admin User**
   Set a username, password, and email for your primary administrator. ([Jenkins][11])
5. **Finalize Instance Configuration**
   Confirm the Jenkins URL (defaults to your EC2 IP) and finish setup. ([Jenkins][3])

---

## Post-Installation (Optional)

* **Enable SSL** via Let’s Encrypt or AWS Certificate Manager behind an Application Load Balancer. ([Jenkins][3])
* **Install Additional Plugins** such as Git, Docker Pipeline, Kubernetes. ([Jenkins][3])
* **Attach IAM Role** for AWS-integrated builds (S3, CodeDeploy). ([Jenkins][3])
* **Set Up Backups** of `/var/lib/jenkins` to preserve job configs and credentials. ([Jenkins][3])

---

## Troubleshooting

* **Jenkins Fails to Start**

  ```bash
  journalctl -u jenkins -b
  ```

  Inspect logs for errors. ([Jenkins][3])
* **Port 8080 Inaccessible**

  * Verify security group inbound rules.
  * Check local firewall (`firewalld`/`ufw`). ([GeeksforGeeks][4])

---




[1]: https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html?utm_source=chatgpt.com "run-instances — AWS CLI 1.40.10 Command Reference"
[2]: https://docs.aws.amazon.com/cli/v1/userguide/cli-services-ec2-instances.html?utm_source=chatgpt.com "Launching, listing, and deleting Amazon EC2 instances in the AWS ..."
[3]: https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/?utm_source=chatgpt.com "Jenkins on AWS"
[4]: https://www.geeksforgeeks.org/creating-ec2-instance-with-aws-cli/?utm_source=chatgpt.com "Creating an EC2 Instance with AWS CLI: A Simple Tutorial"
[5]: https://stackoverflow.com/questions/59430965/aws-how-to-install-java11-on-an-ec2-linux-machine?utm_source=chatgpt.com "java - AWS - How to install java11 on an EC2 Linux machine?"
[6]: https://www.jenkins.io/doc/book/installing/?utm_source=chatgpt.com "Installing Jenkins"
[7]: https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-22-04?utm_source=chatgpt.com "How To Install Jenkins on Ubuntu 22.04 - DigitalOcean"
[8]: https://www.jenkins.io/doc/book/installing/linux/?utm_source=chatgpt.com "Linux - Jenkins"
[9]: https://stackoverflow.com/questions/15227305/what-is-the-default-jenkins-password?utm_source=chatgpt.com "What is the default Jenkins password? - amazon ec2 - Stack Overflow"
[10]: https://www.geeksforgeeks.org/what-is-the-default-jenkins-password/?utm_source=chatgpt.com "What Is The Default Jenkins Password? - GeeksforGeeks"
[11]: https://www.jenkins.io/doc/book/system-administration/admin-password-reset-instructions/?utm_source=chatgpt.com "Reset the Jenkins administrator password"
