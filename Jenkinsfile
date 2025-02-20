pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            yaml '''
    apiVersion: v1
    kind: Pod
    spec:
    containers:
        - name: jdk
          image: docker.io/eclipse-temurin:20.0.1_9-jdk
          command:
              - "/bin/sh"
              - "-c"
              - "echo hola && sleep infinity"
          tty: true
          volumeMounts:
              - name: m2-cache
                mountPath: /root/.m2
        - name: podman
          image: quay.io/containers/podman:v4.5.1
          command:
              - "/bin/sh"
              - "-c"
              - "echo hola && sleep infinity"
          tty: true
          securityContext:
              runAsUser: 0
              privileged: true
        - name: kubectl
          image: docker.io/bitnami/kubectl:1.27.3
          command:
              - "/bin/sh"
              - "-c"
              - "echo hola && sleep infinity"
          tty: true
          securityContext:
              runAsUser: 0
              privileged: true
    '''
        }
    }

    environment {
        DOCKER_REGISTRY = "rsolo719"
        KUBE_NAMESPACE = "training"
        HELM_RELEASE = "training-release"
    }

    stages {
        stage('Build Angular y Spring Boot') {
            steps {
                container('podman') {
                    script {
                        echo 'Building Angular y Spring Boot...'
                        sh 'docker build -t $DOCKER_REGISTRY/training-angular:latest -f ./training-angular/dockerfiles/Dockerfile .'
                        sh 'docker build -t $DOCKER_REGISTRY/training-spring-boot:latest ./training-spring-boot'
                    }
                }
            }
        }

        stage('Push a Docker Hub') {
            steps {
                container('podman') {
                    script {
                        echo 'pushin a Docker Hub...'
                        withDockerRegistry([credentialsId: 'docker-hub-$DOCKER_REGISTRY', url: 'https://hub.docker.com/']) {
                            sh 'docker push $DOCKER_REGISTRY/training-angular:latest'
                            sh 'docker push $DOCKER_REGISTRY/training-spring-boot:latest'
                        }
                    }
                }
            }
        }

        stage('Deploy en Kubernetes con Helm') {
            steps {
                container('kubectl') {
                    script {
                        echo 'Instalando Helm...'
                        sh '''
                        curl https://get.helm.sh/helm-v3.11.3-linux-amd64.tar.gz -o helm.tar.gz
                        tar -zxvf helm.tar.gz
                        mv linux-amd64/helm /usr/local/bin/helm
                        helm version
                        '''
                        echo 'Deployment en Kubernetes...'
                        sh 'kubectl create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -'
                        sh 'helm upgrade --install $HELM_RELEASE ./training-chart -n $KUBE_NAMESPACE'
                    }
                }
            }
        }
    }

    post {
        always {
            node('c11-8g2y61etqeq') {
                container('kubectl') {
                    script {
                        echo 'Eliminando recursos de Kubernetes...'
                        sh 'helm uninstall $HELM_RELEASE -n $KUBE_NAMESPACE || true'
                        sh 'kubectl delete namespace $KUBE_NAMESPACE || true'
                    }
                }
            }
        }
    }
}