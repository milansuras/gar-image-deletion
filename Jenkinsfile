pipeline {
    agent any

    environment {
        GAR_LOCATION = 'asia-south1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317'
        REPOSITORY = 'jenkins-cicd-satck'
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
                            # Get all image digests excluding the one tagged as 'latest'
                            NON_LATEST_IMAGES=\$(gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY} \
                                --format="get(digest,tags)" | grep -v "latest" | awk '{print \$1}' | sort -u)
                            
                            # Delete all non-latest images
                            for digest in \$NON_LATEST_IMAGES; do
                                echo "Deleting image with digest: \$digest"
                                gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${service}@\$digest --quiet || true
                            done
                        """
                    }
                }
            }
        }
    }
}

