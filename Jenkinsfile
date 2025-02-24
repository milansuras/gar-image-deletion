pipeline {
    agent any

    environment {
        GAR_LOCATION = 'asia-south1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317'
        REPOSITORY = 'jenkins-cicd-stack'
    }

    stages {
        stage('Authenticate Docker') {             
            steps {                 
                withCredentials([file(credentialsId: 'gcp-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {                     
                    sh """                         
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}                         
                        gcloud auth configure-docker ${GAR_LOCATION} --quiet                     
                    """                 
                }             
            }         
        }

        stage('Find and Delete Non-Latest Images') {
            steps {
                script {
                    def services = ["java-backend", "python-backend", "react-frontend"]

                    for (service in services) {
                        sh """ 
                            echo "Checking images for service: ${service}"
                            
                            # Get all image digests and their tags
                            gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY} \
                                --format="value(digest,tags)" | while read -r digest tags; do
                                
                                if [[ "\$tags" != *"latest"* ]]; then
                                    echo "Deleting image with digest: \$digest"
                                    gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${service}@\$digest --quiet --delete-tags || true
                                fi
                            done
                        """
                    }
                }
            }
        }
    }
}

