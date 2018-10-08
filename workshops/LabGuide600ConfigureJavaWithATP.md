# Lab 600: Configure Java with ATP
## Introduction

Autonomous Transaction Processing provides all of the performance of the market-leading Oracle Database in an environment that is tuned and optimized for transaction processing workloads. Oracle Autonomous Transaction Processing (or ATP) service provisions in a few minutes and requires very little manual ongoing administration.


ATP provides a TLS 1.2 encrypted secure connectivity for applications. In fact, using a secure encryption wallet is the only way to connect to an ATP service instance, ensuring every connection to your database is secure, regardless how it gets routed.

To **log issues**, click [here](https://github.com/cloudsolutionhubs/autonomous-transaction-processing/issues/new) to go to the github oracle repository issue submission form.

## Objectives

- Learn how to build a linux Java application server and connect it to an Oracle ATP database service

## Required Artifacts

- The following lab requires an Oracle Public Cloud account. You may use your own cloud account, a cloud account that you obtained through a trial, or a training account whose details were given to you by an Oracle instructor.

## Steps

### **STEP 1: Create a Virtual Cloud Network**

Virutal Cloud Network (VCN) is a private network that you set up in the Oracle data centers, with firewall rules and specific types of communication gateways that you can choose to use. A VCN covers a single, contiguous IPv4 CIDR block of your choice. See [Default Components that Come With Your VCN](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/overview.htm#Default). The terms virtual cloud network, VCN, and cloud network are used interchangeably in this documentation. For more information, see [VCNs and Subnets](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingVCNs.htm).

- Login to your Oracle Cloud Infrastructure and Click n **Menu** and select **Network** and **Virtual Cloud Networks**.

![](./images/500/Picture500-12.png)

In order to create a VCN we need to select a Compartment from the List Scope. For this lab we will be selecting **Demo** compartment.

![](./images/500/Picture500-13.png)

- After selecting **Demo** compartment click on Create Virtual Cloud Network to create VCN

![](./images/500/Picture500-14.png)

- THis will bring Create Virtual CLoud Netowrk screen where you will specify the configurations.

![](./images/500/Picture500-15.png)

- Enter the following the the screen

**Create In Compartment**: Select the sandbox compartment, Demo. By default, this field displays your current compartment.
**Name**: Enter a name for your cloud network.
Check on **Create Virtual Cloud Network Plus Related Resources** option. By selecting this option, you will be creating a VCN with only public subnets. The dialog expands to list the items that will be created with your cloud network.

![](./images/500/Picture500-16.png)

- Click on Create Virtual Cloud Network.

A confirmation page displays the details of the cloud network that you just created. The cloud network has the following resources and characteristics (some of which are not listed in the confirmation dialog):
- CIDR block range of 10.0.0.0/16
- An internet gateway
- A route table with a default route rule to enable traffic to and from the internet gateway
- A [default security](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/securitylists.htm#Default) list
- A public subnet in each availability domain

![](./images/500/Picture500-17.png)


### **STEP 2: Provision a linux compute VM to serve as the app server**

- Provision a linux compute VM to serve as the app server. 

- Log into your Oracle Cloud Infrastructure and click on Menu and select Compute and Instances.

![](./images/500/Picture500-1.png)

- Click on Create Instance

![](./images/500/Picture500-2.png)

- In order to create Compute Instance we need to select a Compartment. Select Demo Compartment which we created in Lab100

Enter the following to Create Linux Instance

- **Name**: Enter a frinedly name to identify your linux instance
- **Availability Domain**: Oracle Cloud Infrastructure is hosted in Regions, which are localized geographic areas. Each Region contains three Availability Domains which are isolated and fault-tolerant data centers that can be used to ensure high availability. In the Availability Domain field, select the Availability Domain in which you want to run the instance. For example, scul:PHX-AD-1.
- **Image Compartment**: Select Demo compartment
- **Boot Volume**: Oracle-Provided OS Image
- **Image Operating System**: Oracle Linux 7.5
- **Shape Type**: Virtual Machine
- **Shape**: The shape of an instance determines the number of CPUs, the amount of memory, and how much local storage an instance will have. Shape types with names that start with VM are Virtual Machines, while shape types with names that start with BM are Bare Metal instances. Choose the appropriate shape for your Virtual Machine instance in the Shape field. For example, VM.Standard1.4.
- **Image Version**: Please select the latest version, 2018.09.25-0(latest)
- **SSH Keys**: If the operating system image for your instance uses SSH keys for authentication (for example, Linux instances), then you must provide an SSH public key. To choose a public key file, ensure that Choose SSH Key Files is selected, then click Browse. 

![](./images/500/BrowseSSH.png)

- Choose the public key to upload (for example, id_rsa.pub), then click Open. Note: some operating systems may use a different interface for file selection.

![](./images/500/UploadSSH.png)

If you do not have ssh key pair you can create using command line.

- Open terminal for entering the commands
- At the prompt, enter the following:
```
ssh-keygen -t rsa -N "" -b "2048" -C "key comment" -f path/root_name
```
Where
- **-t rsa**: Use the RSA algorithm
- **-N "passphrase"**: Passphrase to protect the use of the key (like a password). If you don't want to set a passphrase, don't enter anything between the quotes.
- **-b "2048"**: Generate a 2048 bit key.
- **-C "key comment": A name to identify the key.
- **-f path/root_name**: The location where the key pair will be saved and the root name for the files. For example, if you give the root name as id_rsa, the name of the private key will be id_rsa and the public key will be id_rsa.pub.

![](./images/500/GenerateSSH.png)

- **Virtual Cloud Network Compartment**: Select Demo Compartment
- **Virtual Cloud Network**: In the Virtual Cloud Network field, select the Virtual Cloud Network for the instance which we created earlier in this lab.
- **Subnet Compartment**: Select Demo Compartment
- **Subnet**: In the Subnet field, select the subnet to which to add the instance. For example, the public subnet scul:PHX-AD-1.

- Click on **Create Instance** at the bottom 

![](./images/500/CreateLinux1.png)
![](./images/500/CreateLinux2.png)
![](./images/500/CreateLinux3.png)

- While the instance is being created, its state is displayed as "PROVISIONING".

![](./images/500/ProvisioningLinux.png)

- The status changes to "RUNNING" when the instance is fully operational.

![](./images/500/RunningLinux.png)

- Note the public IP of the machine provisioned and ssh into this host and configure it to run node.js on ATP.

### **STEP 3: Install JDK and JDBC drivers **

This is super easy on linux - Oracle Linux's package manager `yum` provides the JDK and another tool we'll need `wget`

To install the JDK and `wget` you can run this command: 
```bash
yum install -y wget java-1.8.0-openjdk
```

The other dependencies for the app; [Oracle's JDBC](), [Picocli](), and [the app itself] can be downloaded from the project's github repository with the `wget` command. 

```bash
wget --content-disposition https://github.com/sblack4/ATP-REST-Java/blob/master/ATPJava.tar.gz?raw=true
wget --content-disposition https://github.com/sblack4/ATP-REST-Java/blob/master/lib.tar.gz?raw=true
```
They are both compressed into `.tar.gz` files, to unzip them we can use the `tar` command. 
```bash
tar xvzf ATPJava.tar.gz
tar xvzf lib.tar.gz
```

All together the commands should be kind of like this. You  can copy and paste this script and run it if you don't feel like copying and pasting there's a quick way to do it which I've included below. 


```bash
# set ARG1
# to the directory where the app will be installed
ARG1="$1"
homedir=~
eval homedir=${homedir}
APP_DIR=${ARG1:-"$homedir"}

# get jdk
yum update && yum install -y java-1.8.0-openjdk.x86_64
yum install -y java-1.8.0-openjdk.x86_64

# make install folder
cd ${APP_DIR}
mkdir ATPJava
cd ATPJava

# include --content-disposition
# because otherwise it names file with params (eg ATPJava.tar.gz?raw=true)
wget --content-disposition https://github.com/sblack4/ATP-REST-Java/blob/master/ATPJava.tar.gz?raw=true
wget --content-disposition https://github.com/sblack4/ATP-REST-Java/blob/master/lib.tar.gz?raw=true

tar xvzf ATPJava.tar.gz
tar xvzf lib.tar.gz

# run it
./run.sh --help
```

To download and run the installer run the below commands. If you want to put the app in a specific folder then 

```bash
curl -O https://raw.githubusercontent.com/sblack4/ATP-REST-Java/master/bin/install.sh
chmod +x install.sh
# /app/directory is optional
# defaults to home directory 
./install.sh [/app/directory]
```
