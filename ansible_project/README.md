# Billbox Ansible Playbook

All ansible playbook, variables and script required to:

- Deploy or Redeploy existing and new containers
- Install all the databases (Mongo & Mysql) needed to run billbox
- Configure and keep up to date all forward & reverse proxy settings
- Configure uptime monitoring of upstream partners
- Configure shipping of logs from proxy instances to rapid7 insightops


Introduction
-------------

The purpose of this documentation is to help establish a working model on how to setup local machine and how to successfully run Billbox safely. This documentation
covers the following environment:

- prod: (Production)
- uat: (User Acceptance Test)
- test: (Testing environment but it's incomplete for now)


## Prerequisite
- have the ansible-vault passwod file
- have access to all or the required ansible role on the git repository
- have access to the ssh key file for the environment in question.(No production ssh is ever shared as its' through AWX as much as possible). This is only required when configuring EC2 instances such as proxy or database boxes.
- install python. Preferably though pyenv in order to freely switch version
- install pyenv-virtualenv to help create python virtual envs to your machine to test different versions of python
- Make sure there is pip package manager
- install using pip
  - ansible-core
  - boto
  - boto3
  - awscli

```shell
$ python --version
  Python 3.8.12 (9ef55f6fc369, Oct 25 2021, 05:10:01)
  [PyPy 7.3.7 with GCC Apple LLVM 13.0.0 (clang-1300.0.29.3)]

$ pyenv versions
  system
  3.10.2
  3.8.12
  dev-pypy-3.8
* pypy3.8-7.3.7 (set by /Users/joseph/.pyenv/version)
  pypy3.8-7.3.7/envs/dev-pypy-3.8

$ pip install ansible-core==2.12.2 boto3 boto awscli

$ pip freeze | grep -e ansible-core -e boto3 -e boto -e awscli
  ansible-core==2.12.2
  awscli==1.22.55
  boto==2.49.0
  boto3==1.21.0
  botocore==1.24.0

```

## How to run a playbook

- After checking out the code from the git repository
- create top level folder localrepo/main/billbox/configuration
- while at the top level of the project run the following

```shell
$ ansible-galaxy role install -r roles/requirements.yml
$ ansible-galaxy collection install -r roles/requirements.yml
```

Once those are done you can run a playbook with the following command:

#### RUN playbook on EC2 instances

```shell
AWS_PROFILE=<awscli_profile_name> ansible-playbook -i ~/path/to/project/projectname/inventory -u ubuntu --private-key ~/path/to/sshkeyfile/folder/billbox/GH/GHBillboxUATOps.pem ~/Dreamoval/Engineering/infrastructure_as_code/ansible_projects/projectname/all_playbook.yml --vault-password-file ~/path/to/vault/passwodfile/projectname/uat/vault.password  -e "global_var_environment=uat" --skip-tags facts
```

#### RUN playbook about containers only

```shell
AWS_PROFILE=<awscli_profile_name> ansible-playbook -i ~/path/to/project/projectname/inventory ~/Dreamoval/Engineering/infrastructure_as_code/ansible_projects/projectname/all_playbook.yml --vault-password-file ~/path/to/vault/passwodfile/projectname/uat/vault.password  -e "global_var_environment=uat" --skip-tags facts
```

## Folder Structure
```
/ansible_project_billbox
    |
    +-- collections
          - amazon
          - community
    |
    +-- files
    |
    +-- inventory
          - group_vars
    |
    +-- inventory_backup
          - group_vars
    |
    +-- playbook_vars
          - prod
          - test
          - uat
    |
    +-- roles
          - elasticbeats
          - mongo_family
          - monitoring_family
          - networking
          - percona_mysql
          - security
          - util
          - webserver_family
    |
    +-- templates
```

## Main Config file
The ansible.cfg is the main configuration file where important ansible settings are defined. These settings depict how ansible should behave when executing the playbooks.

<br>

### Playbook Roles
There are a number of Roles that install and prepare the instances for service deployments. Each role executes a series of tasks that install packages and configure the instances. The roles are listed below with their descriptions.

| role | description |
| :----| :-----------|
| ansible_role_utils | installs base utilities, os packages, role dependency packages, and configures them|
| ansible_role_mongo_family | installs and configures all mongo databases|
| ansible_role_security | sets up inventory security with host authentication, firewall, nginx and hardens the inventory OS |
| java_family | installs and configures java requirements on the inventory|
| ansible_role_networking | sets up inventory's networking (Ip tables, proxy and os level firewall)|
| ansible_role_percona_mysql | installs and configures other database servers with percona mysql|
| ansible_role_monitoring_family | installs and configures datadog and insightOps monitoring agents either on the ec2 instances or as sidecars to ecs containers|
| ansible_role_webserver_family | installs webserver packages and configures  webservers on hosts|
| ansible_role_elasticbeats | installs and configures elasticbeats for data transfer|
| collections | downloads and installs collections from the ansible galaxy repository|

<br>

### Playbook Variables
All playbooks make heavy use of variables. Each service playbook has a dedicated variable file where all required variables are defined. Playbook variables documentation can be found [here](https://github.com/DreamOval/ansible_project_billbox/tree/develop/playbook_vars#playbook-variables-documentation).

### Playbook Inventory
The inventory specifies the target hosts that ansible connects to. The inventory group vars also specify ssh arguments for the hosts.

<br>

### EC2 and RDS inventory
  * The bb.aws_ec2.yml and bb.aws_rds.yml files are the main inventory files for all ec2 and rds instances. They define:
    * How we use the **amazon.aws.aws_ec2** plugin to get an inventory of all ec2 and rds instances.
    * The AWS_Profile name we use to connect to AWS services.
    * The regions where the ec2 and rds instances were created.
    * The tags and group names used on the ec2 and rds instances for easy identification.
    * The hostnames and ip_addresses (puplic and private) of all ec2 instances.
    * Databse clusters for rds instances.

<br>

### Inventory Backup.
* A template EC2 inventory script is kept in the inventory_backup directory.
* It is meant to be a backup inventory script that would retrieve information about the ec2 instances.

<br>

## Playbooks
Each billbox microservice has a dedicated playbook that defines a series of actions to deploy or redeploy containers or EC2 instances. All playbooks follow an order for consistency and easy readability. The various sections of the billbox microservice playbooks are detailed below.


## Playbook Structure
The playbooks follow a structure of executing tasks. Since the services run as microservices, their underlying infrastructure are quite similar to each other. Thus, the tasks to create them have certain sections in common. The table below is a breakdown of the common and unique sections as well as their descriptions.

## **First Time Setup**
To successfully configure ansible and run the script on your computer for the first time, follow the laid down steps below;
### Git Configuration ###

```
$ git config --global user.name "<git_username>"
$ git config --global user.email "<your_email>"
$ git config --global core.editor "<preferred_code_editor>"
```
### AWS CLI Setup
```
$ aws configure
### setup credentials for other aws account profiles by editing the aws credentials file.
$ vi ~./aws/credentials
```

### Sdkman Setup
```
### Download and install SDKMan and do the following;
$ sdk list java
$ sdk install <java_version>
$ sdk install gradle
$ sdk install maven
$ sdk install springboot
```

### Ansible Setup
```
  ### Set ANSIBLE_HOME ###
$ export DO_ANSIBLE_HOME=~/<path/to/ansible/home>

  ### Run ansible script test
$ AWS_PROFILE=<your_aws_profile> aws ec2 describe-instances --region eu-west-1
```


### GitLab Access Error
Error sample:
```
\nfatal: unable to access 'https://bitbucket.org/gyan/.git': The requested URL returned error: 400\n"
```
Reason: User not granted access to git repository. To resolve, request for gitlab access credentials.

### Issues installing Spring Cloud on SpringBoot CLI
Error sample:
```
"stderr": "/bin/sh: 1: spring: not found", "stderr_lines": ["/bin/sh: 1: spring: not found"]
```
**Reason**: This usually happens when springboot is not installed or setup on the device. To resolve,
* Open your terminal and follow the steps below;
```
$ apt-get update
$ apt-get upgrade
$ apt-get install unzip zip openjdk-11-jdk ( You can use the latest stable version of jdk)
```
* Open your **~/.bashrc** file and add the command below to the last line of the file and save the new changes.
```
export JAVA_HOME="/usr/lib/jvm/java-1.11.0-openjdk-amd64"
```
* Visit "https://get.sdkman.io" to download and install SdkMan from the official website.
* In your terminal, run
```
$ sdk install maven
$ sdk install springboot 2.3.6.RELEASE
$ sdk use springboot 2.3.6.RELEASE
$ spring install org.springframework.cloud:spring-cloud-cli:2.1.0.RELEASE
```
Once the steps have been executed and all dependencies have been resolved, the playbooks should run just fine.




