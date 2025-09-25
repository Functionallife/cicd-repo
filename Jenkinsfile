pipeline {
    agent any

    environment {
        // GitHub repo (for reference, not needed in checkout scm)
        GIT_URL = "https://github.com/Functionallife/cicd-repo.git"

        // Branch-to-Project mappings
        DEV_PROJECT  = "dev-project-473207"
        UAT_PROJECT  = "uat-project-473207"
        PROD_PROJECT = "micro-answer-469207-r4"

        // VM details (adjust to your infra)
        VM_NAME_DEV  = "jenkins-vm"
        VM_NAME_UAT  = "uat-vm"
        VM_NAME_PROD = "prod-vm"
        ZONE         = "asia-south1-b"
    }

    stages {
        stage('Checkout') {
            steps {
                // Multibranch pipeline auto-checks out the correct branch
                checkout scm
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
                          gcloud compute scp ./index.html ${VM_NAME_DEV}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME_DEV} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else if (env.BRANCH_NAME == "uat") {
                        echo "Deploying to UAT project..."
                        sh """
                          gcloud config set project ${UAT_PROJECT}
                          gcloud compute scp ./index.html ${VM_NAME_UAT}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME_UAT} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else if (env.BRANCH_NAME == "main") {
                        echo "Deploying to PROD project..."
                        sh """
                          gcloud config set project ${PROD_PROJECT}
                          gcloud compute scp ./index.html ${VM_NAME_PROD}:~/ --zone=${ZONE}
                          gcloud compute ssh ${VM_NAME_PROD} --zone=${ZONE} --command "sudo mv ~/index.html /var/www/html/index.html"
                        """
                    } else {
                        echo "Branch ${env.BRANCH_NAME} not configured for deployment."
                    }
                }
            }
        }
    }
}
