pipeline {

    agent any
    
    environment { 
        TAG = sh (returnStdout: true, script: 'date "+%d%m%Y-%H%M%S"').trim()
    }

    stages {        
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
                withCredentials([usernamePassword(credentialsId: 'credenciales_dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
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
                withCredentials([file(credentialsId: 'gcp_credentials', variable: 'GC_KEY')]) {
                    withEnv(["KUBECONFIG=/var/lib/jenkins/.kube/config"]) {
                      sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
                      sh("envsubst < k8s/manifest.yaml | kubectl apply -f -")
                    }
                }
            }
        }
    }
}