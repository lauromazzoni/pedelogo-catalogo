pipeline {
    //agent: define quem vai executar essa rotina
    agent any

    stages {

        stage('Get Source') {
            steps {
                git url: 'https://github.com/lauromazzoni/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    dockerapp = docker.build("lauromazzoni/api-produto:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubecloud'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }

            steps {
                withKubeConfig([credentialsId: 'kube']) {
                    sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                    sh 'chmod u+x ./kubectl'                    
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/deployment.yaml'
                    sh './kubectl apply -f ./k8s/ -R'
                }                
            }
        }
    }
}