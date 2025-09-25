pipeline {
    agent any

    environment {
        // GitHub repo
        GIT_URL = "https://github.com/Functionallife/cicd-repo.git"

        // Branch-to-Project mappings
        DEV_PROJECT  = "dev-project-473207"
        UAT_PROJECT  = "uat-project-473207"
        PROD_PROJECT = "prod-project-473207"

        // VM details (adjust to your infra)
        VM_NAME = "jenkins-vm"
        ZONE    = "asia-south1-b"
    }

    stages {
        stage('Checkout') {
            steps {
                // Use Jenkins GitHub credentials
                git branch: "${env.BRANCH_NAME}",
                    url: "${GIT_URL}",
                  //  credentialsId: 'github-creds'
                echo "Checked out branch: ${env.BRANCH_NAME}"
            }
        }

        stage('GCP Auth') {
            steps {
                // Activate GCP service account stored in Jenkins credentials
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh """
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud auth list
                    """
                }
            }
        }

        stage('Build') {
            steps {
                echo "Running build for branch: ${env.BRANCH_NAME}"
                sh "ls -l"   // replace with real build commands
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == "dev") {
                        echo "Deploying to DEV project..."
                        sh """
                          gcloud config set project ${DEV_PROJECT}
                          gcloud compute scp -r ./index.html ${VM_NAME}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else if (env.BRANCH_NAME == "uat") {
                        echo "Deploying to UAT project..."
                        sh """
                          gcloud config set project ${UAT_PROJECT}
                          gcloud compute scp -r ./index.html ${VM_NAME}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else if (env.BRANCH_NAME == "main") {
                        echo "Deploying to PROD project..."
                        sh """
                          gcloud config set project ${PROD_PROJECT}
                          gcloud compute scp -r ./index.html ${VM_NAME}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else {
                        echo "Branch ${env.BRANCH_NAME} not configured for deployment."
                    }
                }
            }
        }
    }
}
