#TODO: DESCRIPTION

- Create EC2 instance
  #TODO: Spcification

https://www.jenkins.io/doc/book/installing/linux/
- Connect to instance
  sudo apt update
  sudo apt-get update

  install java
    - This is a jenkins dependency
  ```sudo apt install openjdk-17-jre```

  install venv for python 3.10.
    -This is used to create virtual python enviroment and run the pip commands for the test and build of the flask application.
    - This will also install python 3.10
    ```sudo apt-get install python3.10-venv```

  install jenkins
  - This downloads and installs jenkins
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

  - Retrieve admin password from /var/jenkins_home/secrets/initalAdminPassword
 
  - navigate to {public.ip}:8080 on your browser to configure the dashboard. You will be prompted for the admin password
    - We are using admin for simplicty but this shouldn't be done in a scenario with stakes
    - Install the quickstart jenkins plugins

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
  - Hover over your builds numer under the 'Build History' section. Click on 'Console Output'
  - Look for the output directory of the zip compression command

