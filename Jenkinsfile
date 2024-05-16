pipeline {
    agent any
    environment {
        IMAGE = 'gkeapplication'
        TAG = "${BUILD_NUMBER}"
        PROJECT_ID = 'certain-router-423311-c7'
        CLUSTER_NAME = 'sanjay-test'
        LOCATION = 'us-central1'
        CREDENTIALS_ID = 'dff5a71148561c1e194bc182a4071bb5ff83d6b6'
        HELM_CHART_PATH = 'swiggy-app/'
        HELM_RELEASE_NAME = 'swiggy'
        HELM_NAMESPACE = 'swiggy'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Sanjaypramod/GCP-deployments.git'
            }
        }
        stage("Docker Build"){
            steps{
                script{
                    withDockerRegistry(credentialsId: '7ceeac8b2456f423fa0dad3af98354388480093b3', toolName: 'docker'){   
                        sh "sudo docker build -t ${IMAGE} ."
                    }
                }
            }
        }

        stage(' Trivy Scan') {
            steps {
                //  trivy output template 
                sh 'sudo curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'

                // Scan all vuln levels
                sh 'sudo mkdir -p reports'
                sh 'sudo trivy image --format template --template @./html.tpl -o reports/trivy-report.html ${IMAGE}:latest'
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'trivy-report.html',
                    reportName: 'Trivy Scan',
                    reportTitles: 'Trivy Scan'
                ]

                // Scan again and fail on CRITICAL vulns
                // sh 'trivy image --ignore-unfixed --vuln-type os,library --exit-code 1 --severity CRITICAL ${IMAGE}:latest '

            }
        }
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: '7ceeac8b2456f423fa0dad3af98354388480093b', toolName: 'docker'){   
                        sh "sudo docker tag ${IMAGE} sanjay/${IMAGE}:${TAG} "
                        sh "sudo docker push sanjay/${IMAGE}:${TAG} "
                        sh "sudo docker tag ${IMAGE} sanjay/${IMAGE}:latest "
                        sh "sudo docker push sanjay/${IMAGE}:latest "
                    }
                }
            }
        }
        stage("Docker Clean up "){
            steps{
                 sh 'echo " cleaning Docker Images"'
                 sh 'sudo docker rmi -f \$(sudo docker images -q)'
            }
        }
        stage("Helm install "){
            steps{
                 sh "curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3"
                 sh "chmod 700 get_helm.sh"
                 sh "./get_helm.sh"
            }
        }
        stage('GKE Authentication') {
            steps{
                    sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${LOCATION} --project ${PROJECT_ID}"
                
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    sh 'kubectl create ns ${HELM_NAMESPACE} || echo "namespace already created" '
                    sh "helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} --set image.tag=${TAG} --namespace=${HELM_NAMESPACE} --wait"
                }
            }
        }
        
    }
}
