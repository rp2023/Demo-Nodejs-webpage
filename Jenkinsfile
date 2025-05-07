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
