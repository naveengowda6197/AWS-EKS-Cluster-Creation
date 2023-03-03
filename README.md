
How To Create Kubernates Cluster On AWS EKS:-
---------------------------------------------

Before creating cluster activate region in AWS settings
Following steps: 
	Search account settings
	Click on settings
	Activate your region token security
(If above step is not done cluster will create but will get error when creating managed workernodes in cloud stack)

—————————————————————————————————————————————-
NOTE:-
Create a separate EC2 small machine for running Jenkins for deploying production app by triggering Jenkins script on this EC2
and this machine will be used for checking the cluster resource.
(This is not a master node in AWS EKS constrol plane will manage cluster health)

Steps to create Cluster:-
Create EC2 instance
Connect to EC2 Instance
Install JDK on AWS EC2 Instance
Install and Setup Jenkins
Update visudo and assign administrative privileges to Jenkins user
Install Docker
Install and Setup AWS CLI
Install and Setup Kubectl
Install and Setup eksctl
Create eks cluster using eksctl
—————————————————————————————————————————————-

1.  Steps to create and setup an EC2 instance for creating and maintaining whole cluster:-
    launch the ec2 instance with t2 small ubuntu server 20.04. 
    Note: Make sure use new key pair and keep one place as safe
    *Security group add custom TCP rule 8080 for any IP address.

2   Connect to EC2 Instance
    1. Make sure you are in same directory where key file reside.
    2. Give proper permission for file.
    3. Select the instance and click the connect button and copy the SSH command. 
    4 .Login with that SSH command and key file. 


3.   Install JDK and Maven on AWS EC2 Instance(Steps for installing jenkins)
      		
		sudo apt-get update
		
      		sudo apt install openjdk-11-jre-headless
		
      [Note:Install java 11 for Jenkins . Jenkins supported for java 11
      After configuring Jenkins change java version by 
      Later------ sudo update-alternatives --config java]

      sudo apt update
      sudo apt install maven

Verify both installation:
java -version 
mvn -version

4.     Install Jenkins

    Download Jenkins to EC2:
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'


    update the package manager:
    sudo apt-get update 
    sudo apt-get install jenkins


    If Installed successfully should get msg Like bellow:
    jenkins.service - LSB: Start Jenkins at boot time
     Loaded: loaded (/etc/init.d/jenkins; generated)
     Active: active (exited) since Tue 2021-06-22 20:31:18 UTC; 37s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 16297 ExecStart=/etc/init.d/jenkins start (code=exited, status=0/SUCCESS)

    look for Public IPv4 address

http://url/login?from=%2F replaced with EC2 public IP in this url.

Unlock jenkins

Get Jenkins Password using below command
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

unlock Jenkins using above password and install suggested plugin. And create first user.
—————————————————————————————————————————————-

Add jenkins user as an administrator and also ass NOPASSWORD so that during the pipeline run it will not ask for root password.

Open the file /etc/sudoers in vi mode using below command:

sudo vi /etc/sudoers 

Add the following line at the end of the file:
jenkins ALL=(ALL) NOPASSWD: ALL 

After adding the line save and quit the file.(:wq!)

Now use Jenkins as root user run the following command –

sudo su - jenkins  

—————————————————————————————————————————————-

5.    Install Docker

      sudo apt install docker.io

      verify installation

      docker –version

      Add jenkins user to Docker group

      After this Jenkins will be accessing the Docker for building the application Docker images, so  add the Jenkins user to the docker group.

      sudo usermod -aG docker jenkins

—————————————————————————————————————————————-
6. Install and Setup AWS CLI


      sudo apt install awscli

7. verify installation

      aws --version

8. Configure AWS CLI

      After installing the AWS CLI, configure the AWS CLI so that it can authenticate and communicate with the AWS environment.

      configure the AWS –

            aws configure 
      Provide info:
      AWS Access Key ID [None]: add root IAM user credentails
      AWS Secret Access Key [None]: add root IAM user credentails
      Default region name [None]: eu-central-1
      Default output format [None]:leave blank
—————————————————————————————————————————————-
9.  Install and Setup Kubectl

    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.5/bin/linux/amd64/kubectl

     chmod +x ./kubectl

    sudo mv ./kubectl /usr/local/bin

10. Verify the kubectl installation
    kubectl version

—————————————————————————————————————————————-
11.  Install and Setup eksctl

      curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

      sudo mv /tmp/eksctl /usr/local/bin

       Verify the installation by running the command –

      eksctl version
—————————————————————————————————————————————-
—————————————————————————————————————————————-


12. Create eks cluster using eksctl

In all the previous steps we were preparing our AWS environment. Now in this step, we are going to create EKS cluster using eksctl

You need the following in order to run the eksctl command

Name of the cluster : –naveen
Version of Kubernetes : –version 1.17
Region : –name eu-central-1
Nodegroup name/worker nodes : example-cluster-nodegroup
Node Type : t2.medium
Number of nodes: -nodes 2

Here is the eksctl command –

— Create cluster with Node level autoscaling—
eksctl create cluster --name naveen --region eu-central-1 --nodegroup-name example-cluster-nodegroup --node-type t2.medium --nodes-min 2 --nodes-max 4


Note - Wait for some time above because it may take 20-30 minutes to complete and create cluster
Can tract progress in EKS cloud formation.
