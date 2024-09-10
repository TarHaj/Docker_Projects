# Pull and build from this docker hub image
FROM centos:latest

# CentOS reached End Of Life (EOL) so need to change mirror: https://techglimpse.com/failed-metadata-repo-appstream-centos-8/
RUN cd /etc/yum.repos.d/ 
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
RUN yum -y update

# Terraform installation
RUN yum install -y yum-utils
RUN yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
RUN yum -y install terraform

# Run the command terraform version to check the image was correctly installed
CMD ["terraform version"]
