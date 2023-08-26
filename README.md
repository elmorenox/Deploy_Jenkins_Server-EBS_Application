**This repository is used to rehearse:**
  1. The deployment of an AWS EC2 instance.
  2. Installing a Jenkins service on the EC2 instance.
  3. Checking out the Flask application from this repository to test and build through the Jenkins service. 
  4. Deploying the bundle produces by the Jenkins build as an Elastic Beanstalk Application.

**AWS EC2**
- Create EC2 instance
  #TODO: Spcification

**Jenkins**
Install Jenkins
https://www.jenkins.io/doc/book/installing/linux/

  - Acces your EC2 instance's terminal
  - Update apt and apt-get
    sudo apt update
    sudo apt-get update

  Install java
    - This is a jenkins dependency
  ```sudo apt install openjdk-17-jre```

  Install venv for python 3.10.
    -This is used to create virtual python enviroment and run the pip commands for the test and build of the flask application.
    - This will also install python 3.10
    ```sudo apt-get install python3.10-venv```

  Install jenkins
    - These curl and apt-get commands downloads and installs jenkins
    
    ```
      curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
        https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
        /etc/apt/sources.list.d/jenkins.list > /dev/null
    
      sudo apt-get install jenkins
    ```
    
  Start jenkins
    ```sudo systemctl start jenkins```

  Accessing the Jenkins dashboard
    - Retrieve admin password from /var/jenkins_home/secrets/initalAdminPassword
    - Navigate to {public.ip}:8080 on your browser to configure the dashboard. You will be prompted for the admin password
    - We are using admin for simplicty but this probably shouldn't be done in a scenario with stakes
    - You will be prompted to install the reccomended pluging or choose your own. Install the quickstart jenkins plugins.
    - Install the Jenkins 'Pipeline Utility Steps':
      - https://plugins.jenkins.io/pipeline-utility-steps/
      - ![Screenshot 2023-08-25 at 8 48 15 PM](https://github.com/elmorenox/Deploy_Jenkins_Server-EBS_Application/assets/8043346/3532d82d-9d18-472c-9b63-028cd1f932b0)

  **Github tokens**
  - Before you build the pipeline you'll need credentials to authenticate jenkins against Github
    - Nagivate to your Github users setting
    - Cick on 'Developer Settings'
    - Click on the 'Personal Access Tokens' dropdown and select 'Tokens (classic)'
    - Click on the 'Generate new token' dropdown and select 'Generate new token (classic)
    - Add an approriate note like 'Jenkins Controller'
    - You need full control of repo. (If you want to deploy with Jenkins select admin:repo_hook)
    - SAVE THE TOKEN, you wont be presented with the token again. 

  -Create a pipeline
    - Navigate to 'New Item'
    - Name your pipeline
    
  - Definition
    - Pipine script from SCM #TODO: What does this mean?
    - Select Git from the SCM dropdown
    - Add the repositories url to the 'Repository URL' field

  - Credentials continued
   - Navigate to Gitu  
   - Click on 'Add' under the credential drop down, select Jenkins
   - Click on the Kind dropdown select 'Username with password'
   - Add your Github uersname to the username' field
   - In the 'password' field add the GitHub token you generated.
   - Click 'Add'. The modal will close.
   - You can now select your credential in the 'Credentials' dropdown
   - Ensure you have the desired brnach to pull in the 'Branch Specifier' field
   - Click 'Apply' then 'Save'

  -Build
    - Click on 'Build Now'
    - Dowload the output zip to your local machine
    - Hover over your builds numer under the 'Build History' section. Click on 'Console Output'
    - Look for the output directory of the zip compression command. It set ouput in a jenkins workspace folder.



