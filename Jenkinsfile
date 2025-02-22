pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            yaml '''
    apiVersion: v1
    kind: Pod
    metadata:
        labels:
            app: jenkins-agent
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
        IMAGE_ORG = "rsolo719"
        CONTAINER_REGISTRY = "docker.io"
        KUBE_NAMESPACE = "training"
        HELM_RELEASE = "training-release"
        KUBERNETES_CLUSTER_CRED_ID = 'training-config'
        CONTAINER_REGISTRY_CRED = credentials("docker-hub-rsolo719")
    }

    stages {
        stage('Prepare enviroment') {
            steps {
                container('podman') {
                    script {
                        sh '''
                            set -x
                            curl -v docker.io
                            podman login $CONTAINER_REGISTRY -u $CONTAINER_REGISTRY_CRED_USR -p $CONTAINER_REGISTRY_CRED_PSW
                            set +x
                        '''
                        sh '''
                            set -x
                            podman login --get-login docker.io
                            podman --version
                            set +x
                        '''
                    }
                }
                container('kubectl') {
                    script {
                        withKubeConfig([credentialsId: "$KUBERNETES_CLUSTER_CRED_ID"]) {
                            echo 'Instalando Curl y Helm...'
                            sh 'cat /etc/os-release'

                            sh 'apt-get update && apt-get install -y curl || apk add --no-cache curl'

                            sh 'curl https://get.helm.sh/helm-v3.11.3-linux-amd64.tar.gz -o helm.tar.gz'
                            sh 'tar -zxvf helm.tar.gz'
                            sh 'mv linux-amd64/helm /usr/local/bin/helm'

                            sh 'helm version'
                        }
                        
                    }
                }
            }
        }

        stage('Build Angular y Spring Boot') {
            steps {
                container('podman') {
                    script {
                        echo 'Building imágenes de Angular y Spring Boot...'
                        sh 'podman build -t $IMAGE_ORG/training-angular:latest -f ./training-angular/dockerfiles/Dockerfile .'
                        sh 'podman --version'
                        sh 'podman build -t $IMAGE_ORG/training-spring-boot:latest ./training-spring-boot'
                        sh 'podman --version'
                    }
                }
            }
        }

        stage('Push a Docker Hub') {
            steps {
                container('podman') {
                    script {
                        echo 'pushin a Docker Hub...'
                        sh 'podman push $IMAGE_ORG/training-angular:latest'
                        sh 'podman push $IMAGE_ORG/training-spring-boot:latest'
                    }
                }
            }
        }

        stage('Deploy en Kubernetes con Helm') {
            steps {
                container('kubectl') {
                    script {
                        sh 'kubectl create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -'
                        sh 'helm upgrade --install $HELM_RELEASE ./training-chart -n $KUBE_NAMESPACE'
                    }
                }
            }
        }
    }

}