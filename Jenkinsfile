pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    dockerapp = docker.build("erismardeveloper/guia-jenkins:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        
        stage('Deploy no Kubernetes') {
            agent none
            environment {
                TAG_VERSION = "${env.BUILD_ID}"
                K8S_NAMESPACE = 'default'
            }
            steps {
                script {
                    docker.image('bitnami/kubectl:latest').inside('--entrypoint ""') {
                        withKubeConfig([credentialsId: 'kubeconfig']) {
                            sh 'echo "Rodando deploy dentro de um contêiner Docker (bitnami/kubectl)..."'
                            sh 'kubectl version --client'

                            sh 'echo "Atualizando deployment.yaml com a tag: ${TAG_VERSION}"'
                            sh 'sed -i "s/{{tag}}/${TAG_VERSION}/g" ./k8s/deployment.yaml'

                            sh 'echo "Aplicando manifestos Kubernetes..."'
                            sh 'kubectl apply -f ./k8s/deployment.yaml --namespace ${K8S_NAMESPACE}'
                        }
                    }
                }
            }
        }
    }
}