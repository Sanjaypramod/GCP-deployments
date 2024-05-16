pipeline {
    agent {
        label 'ubuntu-latest'
    }

    environment {
        // Define environment variables
        DOCKER_REGISTRY = 'swiggy.azurecr.io'
        IMAGE_REPOSITORY = 'abdulfayisgkeapplication'
        DOCKERFILE_PATH = 'Dockerfile'
        TAG = "${BUILD_ID}"
        CONTAINER_REGISTRY_CREDENTIALS_ID = 'a68374f6-35a2-4766-80ba-eb8f374a1f11'
        KUBERNETES_CREDENTIAL_ID = 'kube-f4yiee'
    }

    stages {
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Login to Docker registry
                    docker.withRegistry("https://${env.DOCKER_REGISTRY}", env.CONTAINER_REGISTRY_CREDENTIALS_ID) {
                        // Build and push Docker image
                        docker.build("${env.DOCKER_REGISTRY}/${env.IMAGE_REPOSITORY}:${env.TAG}", "-f ${env.DOCKERFILE_PATH} .").push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Setup Kubernetes and Helm
                    sh 'helm version --client'
                    sh 'docker version'

                    // Kubernetes and Helm deployment commands
                    kubernetesDeploy(
                        configs: 'kubeconfig',
                        kubeconfigId: env.KUBERNETES_CREDENTIAL_ID,
                        enableConfigSubstitution: true,
                        serverUrl: 'https://your.kubernetes.server'
                    )

                    helm(
                        helmToolName: 'Helm 2.14.1', 
                        tillerNamespace: 'kube-system', 
                        chartName: 'swiggy-app', 
                        releaseName: 'swiggy', 
                        flags: "--namespace swiggy --set image.repository=${env.DOCKER_REGISTRY}/${env.IMAGE_REPOSITORY},image.tag=${env.TAG},image.pullSecret=${env.IMAGE_PULL_SECRET}"
                    )
                }
            }
        }
    }
}

