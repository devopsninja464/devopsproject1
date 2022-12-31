### DevOps Near RealTime Project 

- In this project, we used these below tools to implement the devops near realtime project. 
- Git, Jenkins, NGINX, Ansible, AWS ECR and K8s 

- Overview of the project:

      Developer push the nodejs code to the Github repository (SCM tool).
	  Jenkins will pull the code from the github.
	  Docker will build the nodeJs docker image.
	  Push Docker image to the AWS ECR repository
	  we need to use AWS cli to configure the aws access key and secret key credentials
	  use the Shell script to create the K8s secret object
	  Write the ansible playbook to deploy the k8s cluster
	  
- Installation of Docker:

		   sudo apt install docker.io 
		   sudo apt update
		   sudo apt install docker.io
		   sudo snap install docker
		   sudo usermod -aG docker ubuntu
		   sudo systemctl restart docker
		   sudo systemctl status docker
		   sudo reboot

- Installation of k3s:

		 curl -sfL https://get.k3s.io | sh -  
		 curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl 
		   chmod +x kubectl
		   sudo mv kubectl /usr/local/bin/
		  kubectl version -o yaml
		  sudo k3s kubectl get nodes
		  sudo k3s kubectl get pods

- Installation of Ansible:

		  sudo apt -y install python3-pip python3
		  sudo pip3 install ansible
		  ansible --version
Note: To run the ansible playbook, we need to move the kubeconfig file to the ~/.kube/config location hence copy the k3s service config file to the kube config location. 
		 sudo cp -r /etc/rancher/k3s/k3s.yaml ~/.kube/config
		 mkdir ~/.kube
		 sudo cp -r /etc/rancher/k3s/k3s.yaml ~/.kube/config

reference for more details on k8s ansible module: https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html

kubeconfig param: Path to an existing Kubernetes config file. If not provided, and no other connection options are provided, the Kubernetes client will attempt to load the default configuration file from ~/.kube/config. Can also be specified via K8S_AUTH_KUBECONFIG environment variable.

- Installation of Jenkins:

		wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   		sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
		sudo apt update
		sudo apt install jenkins
		sudo systemctl start jenkins
		journalctl -xeu jenkins.service
		sudo apt install openjdk-11-jdk -y (jenkins need Java11 atleast to work)
		sudo systemctl start jenkins
		sudo systemctl status jenkins

- Installation of NGINX:

		 sudo apt update
		 sudo apt install nginx

Note: we will be using Nginx as a reverse proxy for the the jenkins server to redirect the jenkins 8080 port traffic to the 80. 

here is the configuration changes, 

		sudo vi sites-enabled/default
		Add this block under the server { } block. 
		    location / {
            # pass requests to the Flask host
            proxy_pass http://127.0.0.1:8080;
            proxy_read_timeout  90;
        }
		  sudo systemctl reload nginx.service
		  sudo nginx -t
		  sudo systemctl restart nginx
		  sudo systemctl restart jenkins


Note: we need to add jenkins user to the docker group

 		 sudo usermod -aG docker jenkins 

As jenkins need to run the ansible, we need to provide some permisison to jenkins to communicate with the Ansible
for testing purposes,  jenkins ALL=(ALL) NOPASSWD: ALL added in the /etc/sudoers file. 

- JenkinsFile template:

		 pipeline {
		 agent any 
		 environment
		 {
        VERSION = "${BUILD_NUMBER}"
        IMAGE = 'nodeapp'
      
        ECRCRED = 'ecr:us-east-1:registrycred'
        ECRURL = 'https://279214042703.dkr.ecr.us-east-1.amazonaws.com/nodeapp'
        }   stages {
        stage('GitCheckOut') {
            steps {
                git credentialsId: 'githubcred', url: 'https://github.com/devopsninja464/web_login_automation.git'
            }
        }
         stage('DockerBuild') {
            steps {
                script {
                    dockerImage = docker.build  IMAGE + ":$BUILD_NUMBER"
                }
            }
        }
         stage('DockerPushToECR') {
            steps {
                script {
                    docker.withRegistry(ECRURL, ECRCRED) {
                       dockerImage.push()
                   }
                   }
                }
            }
            stage('Deploy') {
            steps {
                script {
                     sh 'ansible-playbook deploy_node_webapp.yml'
                   }
                   }
                }
            }
			}
		

we used the jenkinsfile to configure the set up, whenever developer pushes the code to the github, jenkins will pull the and build docke rimage and push that image to the AWS ECR repository and then ansible will pull the image through the Kubernetes replicaset and nodejs application wil listen on the k8s service provided nodeport.

-  Lesson Learned:

		 For practice purpose, recommend to use the k3s , Lightweight Kubernetes . https://k3.io . 
		 used k8s cluster or minikube but had issues due to instance size or memory. 
		 Tested above installation and packages, worked fine. 
		 
		 if there are any issues while setting up this project, 
		 please write an email to devopsninja464@gmail.com.  
