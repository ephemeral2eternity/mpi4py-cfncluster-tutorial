# mpi4py in Amazon Web Service EC2
This is a tutorial to use Mac to create an elastic HPC cluster in Amazon EC2 and enable mpi4py

## Prerequisites
#### Windows, Linux, macOS, or Unix
#### Sign up for AWS
1. Open http://aws.amazon.com/.
2. Choose **Create an AWS account**.
3. Follow the online instructions.

#### Create an Amazon EC2 Key Pair
You must have an Amazon Elastic Compute Cloud (Amazon EC2) key pair to connect to the nodes in your cluster over a secure channel using the Secure Shell (SSH) protocol. If you already have a key pair that you want to use, you can skip this step. If you don't have a key pair, follow steps in - [Amazon Web Service Tutorial -- steps to create a key pair](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair).
#### Set up the AWS CLI on your laptop
Different installation steps are needed, depending on your operating system and environment. Available installation methods include an MSI installer, a bundled installer, or pip. The following sections will help you
decide which option to use. Please note the AWS CLI makes API calls to services over HTTPS. Outbound connections on TCP port 443 must be enabled in order to perform calls. Use the information at the following
links to install and configure the AWS CLI:
1. [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)

```Bash
$ pip install awscli --upgrade --user
```
To verify the installation, try
```
$ aws --version
aws-cli/1.11.89 Python/3.6.0 Darwin/17.3.0 botocore/1.5.52
```
2. [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

Access keys consist of an access key ID and secret access key, which are used to sign programmatic requests that you make to AWS. If you don't have access keys, you can create them from the AWS Management Console. We recommend that you use IAM access keys instead of AWS account root user access keys. IAM lets you securely control access to AWS services and resources in your AWS account.

The only time that you can view or download the secret access keys is when you create the keys. You cannot recover them later. However, you can create new access keys at any time. You must also have permissions to perform the required IAM actions. For more information, see Permissions Required to Access IAM Resources in the IAM User Guide.

  * Open the IAM console.
  * In the navigation pane of the console, choose Users.
  * Choose your IAM user name (not the check box).
  * Choose the Security credentials tab and then choose Create access key.

To see the new access key, choose Show. Your credentials will look something like this:
```
Access key ID: AKIAIOSFODNN7EXAMPLE
Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

To download the key pair, choose Download .csv file. Store the keys in a secure location.

Keep the keys confidential in order to protect your AWS account, and never email them. Do not share them outside your organization, even if an inquiry appears to come from AWS or Amazon.com. No one who legitimately represents Amazon will ever ask you for your secret key.


For general use, the aws configure command is the fastest way to set up your AWS CLI installation.

```Bash
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```
The AWS CLI will prompt you for four pieces of information. AWS Access Key ID and AWS Secret Access Key are your account credentials.

#### Install Python & pip on the Launch Computer (macOS)
CfnCluster is written in python and is easily installed with pip (the python installation package). If
python is not already installed on the launch computer, install it from here: python.org. Pip is
Amazon Web Services – Deploy an Elastic HPC Cluster
Page 3 of 14
installed by default when using Python 2 >=2.7.9 or Python 3>=3.4. For more information, visit:
[https://pypi.python.org/pypi/pip/](https://pypi.python.org/pypi/pip/).
1. The existence and version of python can be checked from the command line prompt by typing:
```
$ python --version
```
2. The existence and version of pip can be checked from the command line prompt by typing:
```
$ pip –version
```

## Install Amazon HPC Cluster, CfnCluster
Installing CfnCluster is quick and easy. From the launch computer command-line prompt, type:
```Bash
$ sudo pip install --upgrade cfncluster
```
CfnCluster has now been installed.

## Configure and Launch CfnCluster
#### Create the Base CfnCluster Configure File
CfnCluster customization is specified with the CfnCluster “config” file. The config file is a simple text
file with keyword entries. The config file is created with a CfnCluster configuration tool which creates
a baseline config file for the user.

1.At the prompt, type:

```
$ cfncluster configure
```

2.Accept the defaults for the first three entries by typing “enter” after each of the following lines:

```
Cluster Template [default]:

AWS Access Key ID []:

AWS Secret Access Key ID []:
```
**Note**: It is only necessary to enter your credentials if you have not already configured the AWS CLI or
if you would like to override the credentials used by the AWS CLI or the environment.
3.The next prompt requires the AWS region:
```
Acceptable Values for AWS Region ID:
 us-east-1
 us-west-1
 cn-north-1
 ap-northeast-1
 ap-southeast-2
 sa-east-1
 ap-southeast-1
 ap-northeast-2
 us-west-2
 us-gov-west-1
 ap-south-1
 eu-central-1
 eu-west-1
AWS Region ID [us-west-2]:
```
4.Enter the desired AWS region from the examples provided. A copy and paste of the regions
offered ensures correct syntax. This prompt cannot be left blank.
5.At the prompt: “VPC Name [public]:”, a name is requested for use in the config file. This
name is used to head the VPC section of the config file and is not used anywhere else on AWS
or the cluster. Type enter to accept the default.
6.The next prompt asks for an SSH Key Name and offers acceptable values. Copy and paste the
name of the Amazon EC2 keypair created earlier. Note that you should not include “.pem”
suffix.
7.Select a VPC ID from the VPCs listed. Possible VPC IDs are offered for user selection. Copy and
paste the desired VPC ID you want to use.
8.Select a VPC Subnet ID. Possible Subnet ID’s are offered for user selection. Copy and paste the
desired VPC Subnet ID.
9.With this information, the CfnCluster “configure” command creates the ~/.cfncluster directory
and then writes a simple config file to that directory. This configuration file provides the
specifications for the cfncluster to be launched.
10.Review your new config file if you would like to verify that everything is correct:
```
$ cat ~/.cfncluster/config
```
The config file will look like something like this:

```
[aws]
aws_region_name = us-west-2
[cluster default]
vpc_settings = public
key_name = mysshkey

[vpc public]
master_subnet_id = subnet-0b14ad52
vpc_id = vpc-442b5d21

[global]
update_check = true
sanity_check = true
cluster_template = default
```

#### Customize the CfnCluster Config File
Using your favorite editor, add the following two keyword lines to the bottom of the cluster default
section. These lines can go right after the “key_name= mysshkey” keyword entry.
From your editor type:

```
initial_queue_size = 3
max_queue_size = 3
```

**Note**: These values will result in a cluster with three compute nodes on launch and a maximum of
three nodes in the cluster.
A wide range of capabilities can be configured by modifying the `CfnCluster` config file. For example,
the `placement_group` keyword is used to create an EC2 placement group which can result in better
performance for applications which require the lowest latency possible. For the example in this
tutorial, a placement group is not required. More information on the `CfnCluster` config file is found
here:

[http://cfncluster.readthedocs.io/en/latest/working.html](http://cfncluster.readthedocs.io/en/latest/working.html)

#### Launch CfnCluster
1. To launch your CfnCluster, enter the following at the command line prompt:
```
cfncluster create HelloCluster
```

**Note**: In this example, the cluster name is “HelloCluster”.
Full deployment of the CfnCluster, "HelloCluster”, will take about 20 minutes depending on the
instance types chosen and the size of the EBS volumes selected. Deployment progress can be
monitored from the [AWS CloudFormation console events tab](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active).

2. Verify successful completion of the cluster using the following:
```Bash
Beginning cluster creation for cluster: HelloCluster
Creating stack named: cfncluster-HelloCluster
Status: cfncluster-HelloCluster - CREATE_COMPLETE                               
Output:"MasterPublicIP"="35.169.252.95"
Output:"MasterPrivateIP"="172.31.4.98"
Output:"GangliaPublicURL"="http://35.169.252.95/ganglia/"
Output:"GangliaPrivateURL"="http://172.31.4.98/ganglia/"
```

**Note**: Both the public and private IP address for your cluster are provided. Additionally, a Ganglia
monitoring portal is created by default on the master node and can be used to monitor the cluster.

3. Copy and paste the line the GangliaPublicURL in a browser to see a console view of the cluster.

#### Log onto Your CfnCluster
1. Use the public IP address and the ssh key to log into your cluster. For example:
```Bash
ssh -i ~/.ssh/hpc-us-east1.pem ec2-user@35.169.252.95
```
**Note**: Please use public IP address and key name.

2. Verify the current state of your cluster by typing the following command
```
qstat -f
```
Proceed to the next step.


## Submit and Run a Simple Parallel MPI Job written in C
In this exercise you will submit and run a simple parallel MPI job. First, you create an executable, and
then create the job submittal file. Lastly, you launch the job.

#### Create the mpi_hello_world Executable
1.A shared NFS mount is created for you and is available at `/shared`. The share is an EBS
volume mounted and shared using NFS. Change to this directory:

```
$ cd /shared
```

2.Create a directory called "hw_work":

```
$ mkdir hw_work
```

3.Change to the directory you created:

```
$ cd hw_work
```

Using your favorite editor, copy and paste this text into a file named `mpi_hello_world.c`.

```C
/*A Parallel Hello World Program*/
#include <stdio.h>
#include <mpi.h>
main(int argc, char **argv)
{
 int a, node;
 MPI_Init(&argc,&argv);
 MPI_Comm_rank(MPI_COMM_WORLD, &node);
 for ( a = 1; a < 5; a=a+1){
 printf("Hello World from Node %d\n",node);
 }
 MPI_Finalize();
}
```

4.Compile the code:

```
/usr/lib64/openmpi/bin/mpicxx mpi_hello_world.c -o hw.x
```

You have created the `mpi_hello_world` executable.

#### Create the Job Submittal File
1.Create the job submittal file by copying and pasting the following text into a file called `hw.job`

```
#!/bin/sh
#$ -cwd
#$ -N helloworld
#$ -pe mpi 3
#$ -j y
date
/usr/lib64/openmpi/bin/mpirun ./hw.x > hello_all.out
```

#### Launch the Job
2.Submit the job:

```
$ qsub hw.job
```

3.If you are quick, you can check the status of the job with

```
$ qstat
```
**Note**: Two output files are created when the job is complete: (1) `hello_all.out` and (2)
`hello_world.o1`

4.Display hello_all.out to the screen:

```Bash
$ cat hello_all.out
```
The output file should look something like this:

```Bash
[ec2-user@ip-172-31-2-164 hw_work]$ cat hello_all.out
Hello World from Node 0
Hello World from Node 0
Hello World from Node 0
Hello World from Node 0
Hello World from Node 1
Hello World from Node 1
Hello World from Node 1
Hello World from Node 1
Hello World from Node 2
Hello World from Node 2
Hello World from Node 2
Hello World from Node 2
```

5.Display `hello_world.o1` to the screen:

```Bash
$ cat hello_world.o1
```
The output will look like this:

```
[ec2-user@ip-172-31-2-164 hw_work]$ cat hello_world.o2
Fri Sep 2 04:43:00 UTC 2016
```

6.Log off the master instance. Proceed to the next step.


## Install mpi4py and test the installation
#### Using `pip` to install `mpi4py`
1. Log into the master instance
2. Install via `pip`

```Bash
$ sudo env MPICC=/usr/lib64/openmpi/bin/mpicc pip install mpi4py
```

#### Testing the installation
1.Set the environment

```
export PATH=$PATH:/usr/lib64/openmpi/bin
```

2.Test the installation of `mpi4py`

```Bash
mpiexec -n 5 python -m mpi4py.bench helloworld
```



## Run a sample job written in mpi4py

## References
- [Amazon Web Service Tutorial -- steps to create a key pair](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)

- [Amazon Web Service Tutorial -- Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html)

- [Amazon Web Service Tutorial -- Configure the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

- [Amazon Web Service Tutorial - Deploying an Elastic HPC Cluster](https://github.com/ephemeral2eternity/mpi4py-cfncluster-tutorial/blob/master/deploy-elastic-hpc-cluster_project.pdf)

- [Installation Tutorial for `mpi4py`](http://mpi4py.readthedocs.io/en/stable/install.html)
