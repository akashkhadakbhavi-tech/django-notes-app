pipeline {
    agent any

    stages {
        stage("Clone Code") {
            steps {
                echo "Cloning the code"

                git url: "https://github.com/LondheShubham153/django-notes-app.git",
                    branch: "main"
            }
        }

        stage("Build") {
            steps {
                echo "Building the image"

                sh "docker build -t my-note-app ."
            }
        }

        stage("Push to Docker Hub") {
            steps {
                echo "Pushing the image to Docker Hub"

                withCredentials([
                    usernamePassword(
                        credentialsId: "dockerHub",
                        usernameVariable: "dockerHubUser",
                        passwordVariable: "dockerHubPass"
                    )
                ]) {
                    sh '''
                        set -e

                        echo "$dockerHubPass" | docker login \
                            -u "$dockerHubUser" --password-stdin

                        docker tag my-note-app \
                            "$dockerHubUser/my-note-app:latest"

                        docker push "$dockerHubUser/my-note-app:latest"

                        docker logout
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sh '''
                    echo "Checking Kubernetes manifest files..."
                    pwd
                    ls -l deployment.yaml service.yaml
                '''

                withKubeConfig([
                    credentialsId: "kubernetes",
                    restrictKubeConfigAccess: true
                ]) {
                    sh '''
                        set -e

                        echo "Checking Kubernetes connection..."
                        kubectl cluster-info
                        kubectl get nodes

                        echo "Deploying application..."
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml

                        echo "Checking deployment status..."
                        kubectl get deployments
                        kubectl get pods
                        kubectl get services
                    '''
                }
            }
        }
    }
}
