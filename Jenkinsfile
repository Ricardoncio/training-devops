pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "rsolo719"
        KUBE_NAMESPACE = "training"
        HELM_RELEASE = "training-release"
    }

    stages {
        stage('Build Angular y Spring Boot') {
            steps {
                script {
		    echo "Building Angular y Spring Boot..."
                    sh 'podman build -t $DOCKER_REGISTRY/training-angular:latest f- ./training-angular/dockerfiles/Dockerfile .'
                    sh 'podman build -t $DOCKER_REGISTRY/training-spring-boot:latest ./training-spring-boot'
                }
            }
        }

        stage('Push a Docker Hub') {
            steps {
                script {
		    echo "pushin a Docker Hub..."
                    withDockerRegistry([credentialsId: 'docker-hub-$DOCKER_REGISTRY', url: 'https://hub.docker.com/']) {
                        sh 'podman push $DOCKER_REGISTRY/training-angular:latest'
                        sh 'podman push $DOCKER_REGISTRY/training-spring-boot:latest'
                    }
                }
            }
        }

        stage('Deploy en Kubernetes con Helm') {
            steps {
                script {
		    echo "Deployment en kubernetes..."
                    sh 'kubectl create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -'
                    sh 'helm upgrade --install $HELM_RELEASE ./training-chart -n $KUBE_NAMESPACE'
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Eliminando recursos de Kubernetes..."
                sh "helm uninstall $HELM_RELEASE -n $KUBE_NAMESPACE || true"
                sh "kubectl delete namespace $KUBE_NAMESPACE || true"
            }
        }
    }
}