pipeline {

    agent any
    
    environment { 
        TAG = sh (returnStdout: true, script: 'date "+%d%m%Y-%H%M%S"').trim()
    }

    stages {
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/MII-CC-2024/devops_jenkins_lab",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }        
        stage('Build') {
            steps {
                sh '''
                echo "Building..."
                docker build -t jluisalvarez/flask_app:$TAG .
                '''
            }
        }
        stage('Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "Publishing..."
                        docker login -u="${USERNAME}" -p="${PASSWORD}"
                        docker push jluisalvarez/flask_app:$TAG
                    ''' 
                
                }
            }
        }
        stage('Clean') {
            steps {
                sh '''
                echo "Cleaning..."
                docker rmi jluisalvarez/flask_app:$TAG
                ''' 
                
           }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}