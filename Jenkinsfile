pipeline {
    agent any
    
    environment {
        
        GAR_LOCATION = 'us-central1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317' 
        REPOSITORY = 'jenkins-cicd-stack'
    }
    
    triggers {
        // Run every minute
        cron('* * * * *')
    }
    
     stages {
        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-credentials', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}
                        gcloud auth configure-docker ${GAR_LOCATION} --quiet
                    '''
                }
            }
        }
        stage('Clean All Services') {
            steps {
                script {
                    def services = ['react-todo', 'java-springboot', 'flask']
                    
                    services.each { service ->
                        echo "Processing ${service} folder"
                        
                        // Get list of all images in the service folder
                        def images = sh(
                            script: """
                                gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${service} \
                                --format='get(version)' \
                                --include-tags
                            """,
                            returnStdout: true
                        ).trim()
                        
                        // Process each image
                        images.split('\n').each { imageLine ->
                            if (imageLine) {
                                def hasLatestTag = sh(
                                    script: """
                                        gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${service} \
                                        --filter="version=${imageLine}" \
                                        --format="get(tags)" | grep -q "latest" || echo "no_latest"
                                    """,
                                    returnStdout: true
                                ).trim()
                                
                                // If image doesn't have latest tag, delete it
                                if (hasLatestTag == "no_latest") {
                                    echo "Deleting image without latest tag: ${imageLine}"
                                    sh """
                                        gcloud artifacts docker images delete \
                                        ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${service}@${imageLine} \
                                        --quiet
                                    """
                                } else {
                                    echo "Keeping image with latest tag: ${imageLine}"
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Successfully cleaned up images in all folders"
        }
        failure {
            echo "Failed to clean up images"
        }
    }
}