# ansible-movie-code
Ansible Movie Code, an ansible project in the context of HUA DIT course 'Basic DevOps Concepts and Tools'

<p align="left"> <img src="https://komarev.com/ghpvc/?username=panagiotisbellias&label=Profile%20views&color=0e75b6&style=flat" alt="panagiotisbellias" /> </p>

<h3 align="left">Languages and Tools:</h3>
<p align="left"> <a href="https://azure.microsoft.com/en-in/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/microsoft_azure/microsoft_azure-icon.svg" alt="azure" width="40" height="40"/> </a> <a href="https://www.gnu.org/software/bash/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/gnu_bash/gnu_bash-icon.svg" alt="bash" width="40" height="40"/> </a> <a href="https://www.djangoproject.com/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/django/django-original.svg" alt="django" width="40" height="40"/> </a> <a href="https://www.docker.com/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/docker/docker-original-wordmark.svg" alt="docker" width="40" height="40"/> </a> <a href="https://cloud.google.com" target="_blank"> <img src="https://www.vectorlogo.zone/logos/google_cloud/google_cloud-icon.svg" alt="gcp" width="40" height="40"/> </a> <a href="https://git-scm.com/" target="_blank"> <img src="https://www.vectorlogo.zone/logos/git-scm/git-scm-icon.svg" alt="git" width="40" height="40"/> </a> <a href="https://www.jenkins.io" target="_blank"> <img src="https://www.vectorlogo.zone/logos/jenkins/jenkins-icon.svg" alt="jenkins" width="40" height="40"/> </a> <a href="https://kubernetes.io" target="_blank"> <img src="https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg" alt="kubernetes" width="40" height="40"/> </a> <a href="https://www.linux.org/" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/linux/linux-original.svg" alt="linux" width="40" height="40"/> </a> <a href="https://www.nginx.com" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/nginx/nginx-original.svg" alt="nginx" width="40" height="40"/> </a> <a href="https://www.postgresql.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/postgresql/postgresql-original-wordmark.svg" alt="postgresql" width="40" height="40"/> </a> <a href="https://www.python.org" target="_blank"> <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/python/python-original.svg" alt="python" width="40" height="40"/> </a> </p>

## Connectivity

* Create an inventory file (e.g. hosts.yml) that holds the remote hosts that ansible will handle. The file will have entries look like:
```nano
---
# that is a group entry
<group-name>:
    hosts:
        # this is a host entry
        <vm-name>:
            ansible_host: <DNS Name of VM or public IP>
            ansible_port: 22 # because 22 port is for SSH Connection
            ansible_ssh_user: <username in VM>

# in case you have already ssh connection in the VM using ~/.ssh/config no more info are needed like:
<group-name>:
    hosts:
        <vm-name> # The VM name must be the same as the HostName in the ~/.ssh/config
```
For testing you can use either [vagrant](https://www.vagrantup.com/) using [VirtualBox](https://www.virtualbox.org/) making locally VMs or create some extra VMs in some cloud provider like Gcloud, Azure or Okeanos.

* Run
```bash
ansible -m ping all
```
to check connectivity with declared hosts

## Deployment Support

### Pure Ansible
* Make sure you have configured the inventory file and you are now in the root folder of this project.
* Postgres Installation

[postgres-install.yml](playbooks/postgres-install.yml): This playbook installs all the packages needed, starts the postgresql service and creates a database and a user for our django project according to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to install our service. 
```bash
ansible-playbook -l <group-name> playbooks/postgres-install.yml \
-e PSQL_USER=<username-for-db-user> \
-e PSQL_PASSWD=<password-for-db-user> \
-e PSQL_DB=<name-for-our-database>
```
This operation is done automatically with Jenkins CI/CD Tool and Jenkinsfile.
* Django Project Installation

[django-install.yml](playbooks/django-install.yml): This playbook clones django project code, activates virtual environment, installs all requirements, populate .env variables, makes migrations, starts a gunicorn service and installs and configures an NGINX web server with ssl certificates according to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to deploy our app.
```bash
ansible-playbook -l <group-name> playbooks/django-install.yml \
-e SECRET_KEY='<a-random-string-same-every-time>' \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \ # e.g. postgresql://testuser:pass1234@localhost/demo_db
-e ALLOWED_HOSTS=<your-domain-name-where-app-runs>
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile.

### Ansible & Docker
* Containers For Django App

[django-docker.yml](playbooks/django-docker.yml): This playbook installs docker and docker-compose packages, clones django project code, populate .env variables, and scales up the containers declared in the docker-compose.yml of django project taking build instructions from nonroot.Dockerfile of our app, according also to values passed during execution from the command line. Also we declare explicitly to which group of hosts we want to deploy our app.
```bash
ansible-playbook -l <group-name> playbooks/django-docker.yml \
-e SECRET_KEY='<a-random-string-same-every-time>' \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \ # e.g. postgresql://demouser:pass123@db/deploy_db
-e ALLOWED_HOSTS=<your-domain-name-where-app-runs>
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile.

### Kubernetes Deployment Usage
* The [django-populate-env.yml](playbooks/django-populate-env.yml) is used to populate the .env variables after we have cloned locally our project and before we will be applying the k8s .yaml files to have entities creation. Because a configmap must be generated from .env file we have locally, so the playbook doesn't handle some remote host but only localhost (No need to declare group of hosts). These values are passed from the command line like before.
```bash
ansible-playbook playbooks/django-populate-env.yml \
-e SECRET_KEY='<a-random-string-same-every-time>' \
-e DATABASE_URL=<url-where-database-runs-with-right-credentials> \ # e.g. postgresql://testuser:pass1234@pg-cluster-ip/demo_db
-e ALLOWED_HOSTS=<your-domain-name-where-app-runs>
```
This operation is also done automatically with Jenkins CI/CD Tool and Jenkinsfile with the rest commands needed to deploy on a k8s cluster.

## SSL Configuration using playbooks

* [ansible-https](playbooks/ansible-https.yml): This is used only manually (no Jenkins involved) so as to configure SSL certificates for HTTPS environment in ansible-vm.
* [jenkins-config](playbooks/jenkins-config.yml): This is used so as to configure SSL certificates for HTTPS environment in jenkins-server for extra security.

## files folder

* [django](files/django): has a file that describes how the gunicorn service is built and running in a VM.
* [nginx](files/nginx): has configuration files for nginx sites like the django app and the jenkins service, both in HTTP and HTTPS.

For HTTPS you should make a folder named 'certs' under [files folder](files) and there you have to store (and concatenate according ZeroSSL instructions) your SSL Certificates for your ansible-vm under django subfolder, and your jenkins-server under jenkins subfolder.

## It's our pleasure to contact us at our social media or at github [issues](https://github.com/panagiotisbellias/e-movies-app/issues)