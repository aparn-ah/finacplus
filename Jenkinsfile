pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "aparnahh/finacplus"
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code using SSH key stored in Jenkins
                git branch: 'main',
                    url: 'git@github.com:aparn-ah/finacplus.git',
                    credentialsId: 'github_pat'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_REPO:$BUILD_NUMBER .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'a99d1e4f-0701-4c4c-a65a-833659390b60', 
                                                 usernameVariable: 'USER', 
                                                 passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $DOCKER_HUB_REPO:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'newkubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                        export KUBECONFIG=/var/lib/jenkins/.minikube/profiles/minikube/client.key
                        kubectl set image deployment/finacplus-app finacplus-app=$DOCKER_HUB_REPO:$BUILD_NUMBER --record || kubectl apply -f k8s-deployment.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
