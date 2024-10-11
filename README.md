**Project : Capstone I**

You have been hired as a Sr. DevOps Engineer in Abode Software. They want to
implement DevOps Lifecycle in their company. You have been asked to
implement this lifecycle as fast as possible. Abode Software is a product-based
company and their product is available on this GitHub link.
https://github.com/hshar/website.git
Following are the specifications of the lifecycle:
1. Install the necessary software on the machines using a configuration
management tool
2. Git workflow has to be implemented
3. CodeBuild should automatically be triggered once a commit is made to
master branch or develop branch.
a. If a commit is made to master branch, test and push to prod
b. If a commit is made to develop branch, just test the product, do not
push to prod
4. The code should be containerized with the help of a Dockerfile. The
Dockerfile should be built every time there is a push to GitHub. Use the
following pre-built container for your application: hshar/webapp
The code should reside in '/var/www/html'
5. The above tasks should be defined in a Jenkins Pipeline with the following
jobs:
a. Job1 : build
b. Job2 : test
c. Job3 : prod



**Solution**

Project 1 - DevOps
--------------------------------------------



Project will cover - GITHUB, ANSIBLE, JENKINS & DOCKER


•	Create 3 Machines - Master, Slave1 and Slave2

•	Install configuration management tool on Master – Ansible – Link: https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu

•	Generate a SSH key on Master machine – ssh-keygen

•	Paste and save the public key of master on other Slave machines – cd .ssh – authorized_keys

•	Goto Master and paste the private IPs of slave machines –  sudo nano /etc/ansible/hosts EX. Slave1 ansible_host=172.31.14.9                                                                                                                            Slave2 ansible_host=172.31.5.82

•	Now check the connection between master and Slave machines – ansible -m ping all


**Create a YAML file for executing script files for installation on master and slave machines**

    ---
    - name: task for master
    hosts: localhost
    become: true
    tasks:
    - name: executing script on master
      script: master.sh
      
    - name: task for Slave1
      hosts: Slave1
      become: true
      tasks:
    - name: executing script on Slave1
      script: slave.sh
      
      
      - name: task for Slave2
        hosts: Slave2
        become: true
        tasks:
        - name: executing script on Slave2
          script: slave.sh


**Create 2 script files – Master.sh and Slave.sh**
**

**Master.sh**

    sudo apt update
    sudo apt install openjdk-11-jdk -y
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins –y        


**Slave.sh**
  
    sudo apt update
    sudo apt install openjdk-11-jdk -y
    sudo apt install docker.io -y


•	Check ansible file syntax – ansible-playbook play.yaml --syntax-check

•	Dry Run/Test ansible playbook - ansible-playbook play –check

•	Run Playbook – ansible-playbook play.yaml

•	Open Jenkins - 3.110.124.131:8080 

•	Add nodes in Jenkins – Slave1 and Slave2

•	Through ansible playbook in master install Jenkins/Java

•	Job 1 and 2 will work on develop branch AND job 3 will work on master branch

•	Create GIT workflow

•	Create webHook connection between Jenkins and Github

•	Add nodes in Jenkins – Slave machines 

•	Create Jobs in Jenkins – Job1, Job2, Job3

•	Create – Job1 – Restrict to slave1 

•	Restrict jobs to machines – Ex. Job1 and Job2 – Slave1, Job3 – Slave2

•	Clink on the link provided in the Assignment - https://github.com/hshar/website.git

•	Fork the repository 

•	Create a new file on github – Dockerfile – 

    FROM ubuntu
    RUN apt update
    RUN apt install apache2 -y
    ADD . /var/www/html/
    ENTRYPOINT apachectl -D FOREGROUND 


•	Commit the file – save

•	Create one more branch name – Develop 

•	Go to Jenkins dashboard – Create new – Job1 – freestyle project – Create 

•	Inside click on github project – Add the repository link – Click restrict on slave1

•	Source code management – Paste git repository URL  - Specify branch – Develop

•	Click on git scm polling 

•	Save – Apply 

•	Build

•	Go on slave1 machine and check Job1 created – cd Jenkins/workspace/Job1

•	Go back to Jenkins dashboard – Job1 – Configure – build step – execute shell – 

    Sudo docker build /home/Jenkins/workspace/Job1/ -t imagejob1
    Sudo docker run –itd –p 81:80 –name c1 imagejob1
•	Add – Build now 

•	Copy the public IP of slave1 machine with port no 81 and check

•	Similarly create Job2 and Job3

