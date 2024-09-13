# Build a Terraform Image
This exercise aims at create a docker image where the Provisioning Tool (IaC) Terraform is installed. This container allow the linux user to isolate his Terraform practice from his main local machine.

## PREREQUISITE: 
*  Docker needs to be installed on your local machine

## 1. Alias creation:
If you're working with Docker for a while, you'll want to optimize the command syntax. By default, to run Docker commands, you have to use sudo docker <command>. For example, if you want to pull the CentOS image from Docker Hub (which we'll need later), you'd run this command
````
sudo docker pull centos:latest
````
It will quickly become cumbersome to enter for every command the prefix: sudo docker, so instead lets create an alias, enter the following into your terminal:
````
alias d = 'sudo docker' 
````
To ensure that the alias has been created let's check if the following command displays the images present on your local machine:
````
d images ls
````
WARNING: This alias will remain as long as your terminal window stays open, if you want to permanently store this alias you need to add it to the following file: ~/.bashrc

## 2. Installation of Terraform onto a centOS container:
Creating a image is an iterative process, here the objective is to create an new image using the centOS image where Terraform is installed. 
For best practice I would advise to first create a centOS container where you will install the Terraform package. 

#### 2.1 Creation of the container:
````
##### General command: d run -dit --name <container_name> <image_name>
d run -dit --name centos_test centos
````

#### 2.2 Enter inside your container:
````
#### Standardise command: d exec -it <image_name> bash
d exec -it centos_test
````

#### 2.3 Fix centOS:
It is good practice to always update the linux operating system prior to any installation of other package with the following command:
````
yum install -y yum-utils 
````
If you do so you get the follwoing output:
````
Failed to set locale, defaulting to C.UTF-8
CentOS Linux 8 - AppStream                                                                           0.0  B/s |   0  B     00:00    
Errors during downloading metadata for repository 'appstream':
  - Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8&arch=x86_64&repo=AppStream&infra=container [Could not resolve host: mirrorlist.centos.org]
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8&arch=x86_64&repo=AppStream&infra=container [Could not resolve host: mirrorlist.centos.org]
````
This is a wellknown issue with CentOS 8 which has reached his End Of Life (EOF) so this mirror doesn't have support anymore.
Credit Ramya Santhosh for providing a fix (https://techglimpse.com/failed-metadata-repo-appstream-centos-8/):
For this you will have to alter the filesystem yum package manager mirror repository using the sed command:
````
cd /etc/yum.repos.d
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
````
Then run an update:
````
yum -y update
````
#### 2.4 Install Terraform:
You can find detailed about the installation of terraform on different operating system here (https://developer.hashicorp.com/terraform/install)
On our CentOS we need to run the following command:
````
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y update
yum -y install terraform
````
#### 2.5 Verify that Terraform installation suceed:
Now the last thing we have to do is to verify that the Terraform installation was successful, for that let's just print out the Terraform Version:
````
terraform version
````
If the following pops up:
````
Terraform v1.9.5
````
Well done ! you succesfully installed terraform in a container. Now exit the container:
````
exit
````

## 3. Build a Terraform image using a Dockefile:
Lucky for us the Dockerfile language syntax is one of the easier close to the SQL. 
What is atypical with building a docker image with a Dockerfile is that a Dockefile name can not be altered so with need to isolate it:
#### 3.1 Create a directory and move inside that directory:
````
mkdir Terraform_Image_Build
cd Terraform_Image_Build
````
#### 3.2 Use the editor of your choice for me it will be nvim and create the Dockerfile:
````
vim Dockerfile
````
#### 3.3 Now let's copy paste all the command we have execute to from section 2:
````
cd /etc/yum.repos.d
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y update
yum -y install terraform
````

#### 3.4 It is very easy to update the syntax of this list of command into a Dockerfile format
Just add RUN to all these commands
````
RUN cd /etc/yum.repos.d
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum install -y yum-utils
RUN yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
RUN yum -y update
RUN yum -y install terraform
````
#### 3.5 Image Source & Command:
In this last point we need to specify the origin image to build from and we can add a command to print the terraform version to check if the image was created correctly:
````
FROM centos:latest

RUN cd /etc/yum.repos.d
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

RUN yum install -y yum-utils
RUN yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
RUN yum -y update
RUN yum -y install terraform

CMD ["terraform version"]
````

#### 3.6 Build the image:
All we have left to do is create to build the image using the Dockerfile with the following command:
````
#### Generic command: sudo docker build -t <iamge_name> <path_for_the_image_to_be_created>
d build -t terraform_image .
````

We are all done ! Congrats you know the methodology to create a specific image using a Dockerfile !!
