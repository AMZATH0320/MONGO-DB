pipeline {
    agent {
        label 'j-server'
    }

    environment {
        KUBECONFIG_CREDENTIAL_ID = 'k8s'
    }

    stages {
        stage('Clone DB Repository') {
            steps {
                        sh 'echo "Cloning required code base"'
                        git branch: 'master', url: 'https://github.com/AMZATH0320/MONGO-DB.git'
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    def outputFilePath = "${env.WORKSPACE}/trivy_scan.txt"
                    def docker_image = "mongo:4.4.6"
                    sh "sudo docker pull ${docker_image}"
                    sh "sudo trivy image ${docker_image} > ${outputFilePath}"
                    sh "cat ${outputFilePath}"
                }
            }
        }
        stage('Deploying Mongodb') {
            steps {
                script {
                        withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                            def kubeconfigPath = env.KUBECONFIG
                            sh "export KUBECONFIG=${kubeconfigPath}"
                            //sh "kubectl scale deploy mongodb --replicas=0 -n three-tier"
                            sh 'echo "Viewing DB Manifest to deploy onto k8s cluster"'
                            sh "cat database.yml"
                            sleep(time: 10, unit: 'SECONDS')
                            sh 'echo "Deploying MongoDB Manifest"'
                            sh "kubectl apply -f database.yml --validate=false"
                            sh "kubectl get pods -n three-tier"
                            sh "kubectl get svc -n three-tier"
                            sh "kubectl get secrets -n three-tier"
                        }
                    } 
            }
        }
    }
}
