pipeline {
    agent any

    environment {
        GIT_REPO     = "https://github.com/mohammedzuhair25/deploy-example.git"
        DEPLOYMENT_YAML = "dev/deployment.yaml"
        IMAGE_NAME   = "nginx"
        IMAGE_TAG    = "${BUILD_NUMBER}"   // or use git commit hash
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    sh """
                        sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' ${DEPLOYMENT_YAML}
                    """
                }
            }
        }

        stage('Commit & Push Changes') {
            steps {
                sshagent(['github-ssh-credentials-id']) {
                    sh """
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins CI"
                        git add ${DEPLOYMENT_YAML}
                        git commit -m "Update image to ${IMAGE_NAME}:${IMAGE_TAG} [ci skip]" || echo "No changes to commit"
                        git push origin main
                    """
                }
            }
        }

        stage('Apply to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'k8s-cluster-credentials']) {
                    sh "kubectl apply -f ${DEPLOYMENT_YAML}"
                }
            }
        }
    }
}

