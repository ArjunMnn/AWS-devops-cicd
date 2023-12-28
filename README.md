Prerequisites

Before diving into the tutorial, make sure you have the following:

    AWS account with the necessary permissions.
    Git installed on your local machine.

Step 1: Setting Up CodeCommit
1.1 Create a CodeCommit Repository

Navigate to the AWS Management Console, select CodeCommit, and click "Create Repository." Provide a unique name for your repository.
1.2 Create IAM User and Generate Credentials

Create an IAM user with administrative permissions and generate HTTPS Git credentials for the IAM user.
1.3 Clone Repository Locally

Clone the repository to your local machine using the HTTPS URL and the credentials generated.
1.4 Create Project Files

Inside the local repository, create the following files: index.html, app.js, styles.css. Push the changes to the CodeCommit repository.

bash

# Example Git commands
git add .
git commit -m "Initial commit"
git push origin main

Step 2: Configuring CodeBuild
2.1 Create a Build Project

Open the AWS CodeBuild console and create a new build project. Use the provided buildspec.yaml file as the build specification.

yaml

# buildspec.yaml
version: 0.2
phases:
  install:
    commands:
      - echo Installing Nginx...
      - sudo apt-get update
      - sudo apt-get install nginx -y
  build:
    commands:
      - echo Build started on `date`
      - cp index.html /var/www/html/
      - cp styles.css /var/www/html/
      - cp app.js /var/www/html/
      - cp appspec.yml /var/www/html/
  post_build:
    commands:
      - echo Restarting Nginx...
artifacts:
  files:
    - index.html
    - appspec.yml
    - scripts/**
    - app.js
    - styles.css

2.2 Configure Artifacts

Edit the build project to specify an S3 bucket and a folder for storing artifacts. Create a folder in the S3 bucket named code-output-demo-app.
2.3 Run the Build

Trigger a build in the CodeBuild console. Verify that the build is successful, and artifacts are stored in the S3 bucket.
Step 3: Implementing CodeDeploy
3.1 Create CodeDeploy Application

Navigate to the AWS CodeDeploy console and create a new application named demo-app-application. Specify the compute platform as ec2/on-premises.
3.2 Create Deployment Group

Create a deployment group and associate it with an EC2 instance.
3.3 Install CodeDeploy Agent on EC2 Instance

SSH into the EC2 instance. Execute the provided shell script to install the CodeDeploy agent.

bash

# Install CodeDeploy Agent Script
#!/bin/bash
sudo apt-get update
sudo apt-get install ruby-full ruby-webrick wget -y
cd /tmp
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb
mkdir codedeploy-agent_1.3.2-1902_ubuntu22
# Additional script steps...

3.4 Create appspec.yaml and Scripts

Create an appspec.yaml file with the provided content. Create a scripts folder with install_nginx.sh and start_nginx.sh scripts.

yaml

# appspec.yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/
hooks:
  AfterInstall:
    - location: scripts/install_nginx.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_nginx.sh
      timeout: 300
      runas: root

bash

# scripts/install_nginx.sh
#!/bin/bash
apt-get update
apt-get install nginx -y

bash

# scripts/start_nginx.sh
#!/bin/bash
service nginx start

3.5 Build and Deploy

Trigger another CodeBuild to ensure the latest code artifact is in the S3 bucket. Create a deployment in CodeDeploy, specifying the S3 location of the artifact.
3.6 Configure EC2 Role

Create a role for the EC2 instance with permissions to access S3 and CodeDeploy. Attach the role to the EC2 instance.
3.7 Restart CodeDeploy Agent

On the EC2 instance, restart the CodeDeploy agent.
Step 4: Setting Up CodePipeline
4.1 Configure CodePipeline

Open the AWS CodePipeline console and create a new pipeline. Add CodeCommit as the source, CodeBuild as the build stage, and CodeDeploy as the deploy stage.
4.2 Create Pipeline

Click "Create Pipeline" to initiate the pipeline. The pipeline will automatically fetch code from CodeCommit, build using CodeBuild, and deploy using CodeDeploy.
4.3 Test the Web App

Access the public IP of the EC2 instance. Verify that the Todo List web app is successfully deployed.
Conclusion

Congratulations! You've successfully set up a CI/CD pipeline for deploying a Todo List web app using AWS DevOps tools. This tutorial has covered each step in detail, providing you with a solid foundation for implementing similar projects in the future.
Dive Deeper

For a detailed walkthrough and explanation of each step, visit the accompanying blog post: https://arjunmenon.hashnode.devcicd-pipeline-using-aws-codecommit-codebuild-codedeploy-and-codepipeline