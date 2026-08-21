pipeline {
    agent any
    environment {
        IMAGE_NAME = "hello-app:${BUILD_NUMBER}"
    }
    stages {
        // --- NEW STAGE: The "Loop Breaker" ---
                stage('Check for Skip') {
            steps {
                script {
                    def commitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    
                    if (commitMessage.contains('[skip ci]')) {
                        echo "🚫 Build skipped. Commit message contains [skip ci]"
                        currentBuild.result = 'SUCCESS'  // Sets the build to GREEN
                        return // Gracefully exits the pipeline without error
                    } else {
                        echo "✅ Proceeding with build (no [skip ci] found)"
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Import to containerd') {
            steps {
                sh """
                    docker save ${IMAGE_NAME} -o /tmp/app.tar
                    sudo ctr -n k8s.io images import /tmp/app.tar
                """
            }
        }
        stage('Update GitHub YAML') {
            steps {
                sh """
                    sed -i 's|image: hello-app:.*|image: hello-app:${BUILD_NUMBER}|g' deployment.yaml
                """
                
                withCredentials([gitUsernamePassword(credentialsId: 'github-pat')]) {
                    sh """
                        git config user.email "jenkins@utech-iiot.lk"
                        git config user.name "Jenkins Bot"
                        git add deployment.yaml
                        git commit -m "Update image tag to hello-app:${BUILD_NUMBER} [skip ci]"
                        git push https://github.com/sushenjayasuriya/gitops-test-app.git HEAD:main
                    """
                }
            }
        }
    }
    post {
        always {
            sh "rm -f /tmp/app.tar"
        }
    }
}
