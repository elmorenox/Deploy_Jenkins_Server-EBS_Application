# Rehearsing EC2, Jenkins Service and Elastic Beanstalk

**This repository is used to rehearse:**

1. The deployment of an AWS EC2 instance.
2. Installing a Jenkins service on the EC2 instance.
3. Checking out the Flask application from this repository to test and build through the Jenkins service.
4. Deploying the bundle produces by the Jenkins build as an Elastic Beanstalk Application.

![1693024774121]([image/README/1693024774121.png](https://github.com/elmorenox/Deploy_Jenkins_Server-EBS_Application/blob/main/Jenkins-Elasticbeanstalk-Diagram.png))

## **AWS EC2**

Create EC2 instance to host the Jenkins service

- From the EC2 dashboard click on 'Launch Instance'
- Name your instance
- Select an Ubuntu image. 
- Select and t2.Micro instance type. This is a very small application so we don't need much.
- Select a key pair to be able to login to the instance
- Select a security group or set a security that allows inbound traffic over SSH and HTTP traffict on port 22 and 80 respectively
- Launch the instance

## Jenkins


Access your EC2 instance's terminal through instance connect. We'll set up the linux environment to install Jenkins. 
https://www.jenkins.io/doc/book/installing/linux/

Update dependency managers

```bash
sudo apt update
```

```bash
sudo apt-get update
```

- This is a jenkins dependency

```bash
sudo apt install openjdk-17-jre
```

Install venv for python 3.10.
-This is used to create virtual python environment and run the pip commands for the test and build of the flask application.

- This will also install python 3.10

```bash
sudo apt-get install python3.10-venv
```

Install Jenkins

- These curl and apt-get commands downloads and installs jenkins

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]
https://pkg.jenkins.io/debian-stable binary/ | sudo tee
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get install jenkins
```

Start Jenkins service

```bash
sudo systemctl start jenkins
```

Access the Jenkins dashboard

- Retrieve admin password from /var/jenkins_home/secrets/initialAdminPassword
- Navigate to {public.ip}:8080 on your browser to configure the dashboard. You will be prompted for the admin password
- We are using admin for simplicity but this probably shouldn't be done in a scenario with stakes
- You will be prompted to install the recommended plugin or choose your own. Install the quick start jenkins plugins.
- Install the Jenkins 'Pipeline Utility Steps':
  - [https://plugins.jenkins.io/pipeline-utility-steps/](https://plugins.jenkins.io/pipeline-utility-steps/)
  - ![Screenshot 2023-08-25 at 8 48 15 PM](https://github.com/elmorenox/Deploy_Jenkins_Server-EBS_Application/assets/8043346/3532d82d-9d18-472c-9b63-028cd1f932b0)

### Github tokens

Before you build the pipeline you'll need credentials to authenticate jenkins against Github

- Navigate to your Github users setting
- Click on 'Developer Settings'
- Click on the 'Personal Access Tokens' dropdown and select 'Tokens (classic)'
- Click on the 'Generate new token' dropdown and select 'Generate new token (classic)
- Add an appropriate note like 'Jenkins Controller'
- You need full control of repo. (If you want to deploy with Jenkins select admin:repo_hook)
- SAVE THE TOKEN, you wont be presented with the token again

Create your pipeline

- Navigate to 'New Item'
- Name your pipeline
- Definition

  - Pipeline script from SCM #TODO: What does this mean?
  - Select Git from the SCM dropdown
  - Add the repositories url to the 'Repository URL' field
- Navigate to Github
- Click on 'Add' under the credential drop down, select Jenkins
- Click on the Kind dropdown select 'Username with password'
- Add your Github username to the username' field
- In the 'password' field add the GitHub token you generated.
- Click 'Add'. The modal will close.
- You can now select your credential in the 'Credentials' dropdown
- Ensure you have the desired branch to pull in the 'Branch Specifier' field
- Click 'Apply' then 'Save'

Build

- Click on 'Build Now'
- Download the output zip to your local machine
- Hover over your builds number under the 'Build History' section. Click on 'Console Output'
- Look for the output directory of the zip compression command. It set output in a jenkins workspace folder.

Download the zip file to your local machine. You'll need to have an established SSH connection between you local machine and the Jenkins server. See the next section to set that up.

-Once you have and established ssh connection you can run this to download the zip to your local machine

```bash
scp user@ec2.public.ip:/path/to/zip /local/directory
```

### SSH Connection

If you don't have an SSH private and public keys. Use ``ssh-keygen`` on your local machine to make some keys.

- From ~/ssh/ run:

```bash
ssh-keygen 
```

- Open the generated id_rsa.pub key and copy the key.
- Navigate to ~./ssh/ in your EC2 instance
- Open the authorized_keys file
- Append your public key
- Test your SSH connection

```bash
ssh -T user@ip
```

## AWS IAM and Elastic Beanstalk

Create an IAM role for EC2 and EBS. These roles will be used to service EBS deployment, application and EC2 host

- In the IAM Management Console click on Roles
- In the Roles page click on 'Create role'.

EBS Service Role

- Select 'AWS Service'as the Trusted Entity Type
- Select 'Elastic Beanstalk - Customizable' as the Use Case
- No additional permission are needed so you can click 'Next' into step three, "Name Review and Create
- Add an appropriate Role Name like: aws-elasticbeanstalk-service-role

EC2 Role

- Click on 'Create role' again in the IAM Management Console
- Select 'AWS Service as the Trusted Entity Type
- Select 'EC2' as the Use Case
- Add AWSElasticBeanstalkMulticontainerDocker, AWSElasticBeanstalkWorkerTier, AWSElasticBeanstalkWebTier permission policies.
- Name the role 'Elastic-EC2' and create

Create the EBS deployment and application

- Search for the Elastic Beanstalk Amazon Service
- Click on 'Create application'
- Add an Application name like 'url-shorter'
- The GitHub repository is a Flask Application written with Python 3.9 so select the Python as the Platform and 3.9 as the branch
- In the Application Code section, upload the Zip file you downloaded from the Jenkins server
- You'll need to populate the 'Version Label' with a version of the application e.g v1
- Use the ElasticEC2 role to service the EC2 instances and aws-elasticbeanstalk-service-role to service the EBS environment
- Select a VPC
- Select a desired availability zone
- Select your desired boot device and size like General Purpose SSD and 10GB
- Select an Instance type like T2.Micro
- Submit
