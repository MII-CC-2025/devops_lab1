pipeline {

    agent any

    stages {
        stage("Clone Git Repository") {
            steps {
                git(
                    url: "https://github.com/MII-CC-2025/devops_lab1",
                    branch: "main",
                    changelog: true,
                    poll: true
                )
            }
        }
    
        stage('Build') {
            steps {
                sh 'docker --version'
            }
        }
        stage('Test') {
            steps {
                sh 'java --version'
           }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                        string(credentialsId: 'access_key', variable: 'ACCESS_KEY'),
                        string(credentialsId: 'secret_key', variable: 'SECRET_KEY'),
                        string(credentialsId: 'session_token', variable: 'ACCESS_TOKEN'),
                    ]) {
                        withEnv([
                            "AWS_ACCESS_KEY_ID=${ACCESS_KEY}",
                            "AWS_SECRET_ACCESS_KEY=${SECRET_KEY}",
                            "AWS_SESSION_TOKEN=${ACCESS_TOKEN}",
                            "KUBECONFIG=/var/lib/jenkins/.kube/config"
                        ]) {
                            sh "echo $AWS_ACCESS_KEY_ID"
                            sh 'kubectl version'
                        }
                    }
                }
        }
    }
}
