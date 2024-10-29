Project : Capstone II

You are hired as a DevOps Engineer for Analytics Pvt Ltd. This company is a
product based organization which uses Docker for their containerization needs
within the company. The final product received a lot of traction in the first few
weeks of launch. Now with the increasing demand, the organization needs to
have a platform for automating deployment, scaling and operations of application
containers across clusters of hosts. As a DevOps Engineer, you need to
implement a DevOps lifecycle such that all the requirements are implemented
without any change in the Docker containers in the testing environment.
Up until now, this organization used to follow a monolithic architecture with just 2
developers. The product is present on: https://github.com/hshar/website.git

Following are the specifications of the lifecycle:
1. Git workflow should be implemented. Since the company follows a
monolithic architecture of development, you need to take care of version
control. The release should happen only on the 25th of every month.
2. CodeBuild should be triggered once the commits are made in the master
branch.
3. The code should be containerized with the help of the Dockerfile. The
Dockerfile should be built every time if there is a push to GitHub. Create a
custom Docker image using a Dockerfile.
4. As per the requirement in the production server, you need to use the
Kubernetes cluster and the containerized code from Docker Hub should be
deployed with 2 replicas. Create a NodePort service and configure the
same for port 30008.
5. Create a Jenkins Pipeline script to accomplish the above task.
6. For configuration management of the infrastructure, you need to deploy the
configuration on the servers to install necessary software and
configurations.
7. Using Terraform, accomplish the task of infrastructure creation in the AWS
cloud provider.
Architectural Advice:
Softwares to be installed on the respective machines using configuration
management.

Worker1: Jenkins, Java

Worker2: Docker, Kubernetes

Worker3: Java, Docker, Kubernetes

Worker4: Docker, Kubernetes


![image](https://github.com/user-attachments/assets/0aa98014-8933-41bc-82bf-d8532dca18ef)

![image](https://github.com/user-attachments/assets/a2789f80-c72f-46a9-9572-0e63d065c025)


Solution:

Project – 2 


Using terraform going to create – 3 Machines – Kubernetes Master, Kubernetes S1, Kubernetes S2
All tasks on machines will be done through ansible

Install terraform and Ansible machine which we create Manually.

Terraform file code for creating 3 instances – main.tf

    
     provider "aws" {
     region     = "ap-south-1"
     access_key = "AKIA7T4OJ55NHPA"
     secret_key = "1zWluxGgPQXQ7atC/EZo9DGTVoe3whW"
     }

     resource "aws_instance" "Kubernetes-Master" {
     ami           = "ami-0dee22c13ea7a9a67"
     instance_type = "t2.medium"
     key_name      = "Project"
     tags          = {
     Name          = "Kubernetes Master"
     }
     }
     resource "aws_instance" "Kubernetes-Slave1" {
     ami           = "ami-0dee22c13ea7a9a67"
     instance_type = "t2.micro"
     key_name      = "Project"
     tags          = {
     Name          = "Kubernetes Slave1"
     }
     }
     resource "aws_instance" "Kubernetes-Slave2" {
     ami           = "ami-0dee22c13ea7a9a67"
     instance_type = "t2.micro"
     key_name      = "Project"
     tags          = {
     Name          = "Kubernetes Slave2"
     }
     }


•	Now we will check terraform file – terraform init 

•	After successfully we will do – terraform apply

•	Generate public key in manual machine – ssh-keygen

•	Copy the public ip from – cd .ssh -  id_ed25519.pub

•	Paste the public ip in other machines in – cd .ssh - authorized_keys

•	In manual machine in path – sudo nano /etc/ansible/hosts

•	Copy all the private IP’s of other machines like ex. – 

    [Master]
    Kubernetes-Master ansible_host=172.31.33.205
    [Slave]
    Kubernetes-Slave1 ansible_host=172.31.9.228
    Kubernetes-Slave2 ansible_host=172.31.1.92

Create script file on manual machine – sudo nano machine1.sh

    sudo apt update
    sudo apt install openjdk-11-jdk -y
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins –y

 Create script file for Kubernetes Master –

    sudo apt update
    sudo apt install openjdk-11-jdk -y
    sudo apt install docker.io -y
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo systemctl enable --now kubelet


Create script file for Kubernetes Slaves – 

    sudo apt update
    sudo apt install docker.io -y
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg
    sudo mkdir -p -m 755 /etc/apt/keyrings
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo systemctl enable --now kubelet


Now create playbook – sudo nano play.yaml
   
    ---
    - name: executing script on Machine-1
    become: true
    hosts: localhost
    tasks:
    - name: executing script on Machine-1
    script: machine1.sh
    - name: executing script on Master
    become: true
    hosts: Master
    tasks:
    - name: executing script on Master
    script: K8s-M.sh
    - name: executing script on Slaves
    become: true
    hosts: Slave
    tasks:
    - name: executing script on Slaves
    script: K8s-S.sh

•	Check syntax - ansible-playbook play.yaml --syntax-check

•	Dry run - ansible-playbook play.yaml –check

•	Run File - ansible-playbook play.yaml

•	Goto Kubernetes master and create kubeadm – sudo kubeadm init --apiserver-advertise-address=172.31.33.205

•	Copy token to join with Slave machines – ex.

          kubeadm join 172.31.33.205:6443 --token 16w1xh.jzwdp2cookdrd4jk \
        --discovery-token-ca-cert-hash sha256:837dd517e96a9d8388eed7f01d536919a962383268e9937393aee4824f7aec26


•	Run this commands one by 1 for Kubernetes configuration –  

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config


•	Install network tool – Calico
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml -O
    
•	Apply calico network – kubectl apply –f calico.yaml



•	Now open Jenkins in Local machine created 

•	Add k-master node – Manage node – Add node 

•	Fork the repository 

•	Create a Dockerfile in the github repository

•	Give credentials of DockerHub to Jenkins – 
Goto Jenkins – Manage Jenkins – Credentials – Global Credentials – Dockerhub username and password – create

•	Now create pipeline – pipeline script  for hello – 


     pipeline {
     agent none
     environment {
     DOCKERHUB_CREDENTIALS = credentials("1ea4e523-0305-49fe-a6de-5c88986fb795")
    }

    stages {
    stage('Hello') {
    steps {
                echo 'Hello World'
            }
        }
    }
    }
•	Add file for deployment in git repository – deploy.yaml

•	Add file for service in git repository – service.yaml


•	Create pipeline ---

      pipeline{
      agent none
      environment {
        DOCKERHUB_CREDENTIALS=credentials(‘1ea4e523-0305-49fe-a6de-5c88986fb795’)
      }

     stages{
        stage('Hello'){
            agent{ 
                label 'Kub-master'
            }
            steps{
                echo 'Hello World'
            }
        }
        stage('git'){
            agent{ 
                label 'Kub-master'
            }

            steps{

                git ' https://github.com/atullahase3212/Project-2.git'
            }
        }
        stage('docker') {
            agent { 
                label 'Kub-master'
            }

            steps {

                sh 'sudo docker build /home/ubuntu/jenkins/workspace/project2 -t atul03030304/abcd'
                sh 'sudo echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'sudo docker push atul03030304/abcd'

            }
        }
        stage('Kubernetes') {
            agent { 
                label 'Kub-master'
            }

            steps {

                sh 'sudo kubectl create -f deploy.yaml'
                sh 'sudo kubectl create -f service.yaml'
            }
        }        

    }
    }

     
