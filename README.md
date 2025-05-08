# Demo-Nodejs-webpage
My new project

Steps-01 Jekins Setup
=======================
Create AWS EC2 machine with t2.medium(20GB) & connect with gitbash 

install java
========
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

Install Jenkins
========

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

Start jenkins
===============
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins


Step-02 Install docker in Jenkins ec2
======================================
sudo apt update
curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker ubuntu 
exit


step-03 Open jenkins server in browser using VM public ip
=====================================================
http://public-ip:8080/


step-04 Copy jenkins admin paswd
=========================
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

step-05 Create CICD pipeline
=========================
 1) Install required pluging as follow:
    - OWASP Plugin(Open Web Application Security Project).
    - docker: docker compose build, docker pipeline,docker plugin,docker build step.
    - NodeJS plugin
2) Configure maven,docker,dependency check, Nodejs as global tool.

step-06 Create new Item (Nodejs job/project)
==========================================
  - Click on discard old build (Max # of builds to keep "2").


pipeline {
    agent any
    
    tools{
        nodejs "NodeJS-24"
    }
    stages {
        stage('Gitcheckout') {
            steps {
               git branch: 'main', changelog: false, poll: false, url: 'https://github.com/rp2023/Demo-Nodejs-webpage.git'
            }
        }
         stage('Install Dependencies') {
            steps {
              sh "npm install"
            }
        }
         stage('OWASP Dependencies check') {
            steps {
              dependencyCheck additionalArguments: '''--scan ./ --format HTML''', odcInstallation: 'Dp-Check'
            }
        }
         stage('Docker Build & Push') {
            steps {
              script {
                  withDockerRegistry(credentialsId: '23c20c9f-a1d5-4f93-bdaf-626264c2d63b', toolName: 'Docker') {
                      sh "docker build -t demodockerjs ."
                      sh "docker tag demodockerjs rahuldev417/nodejs:latest"
                      sh "docker push rahuldev417/nodejs:latest"
                 }
              }
            }
        }
         stage('Docker Deploy') {
            steps {
              script {
                  withDockerRegistry(credentialsId: '23c20c9f-a1d5-4f93-bdaf-626264c2d63b', toolName: 'Docker') {
                      sh "docker run -d --name demo-nodejs -p 8081:8081 rahuldev417/nodejs:latest"
                 }
              }
            }
        }
    }
}

===================##############=======================

Note: use snipest generator for
Git checkout use git:Git
Docker build & push: withDockerRegistry:sets up docker registry endpoint.(username and password and generate script)
Dependency check: Dependency check:invoke dependancy check.(set arguments and generate script)



