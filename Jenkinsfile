pipeline {
    agent any

    environment {
        GAR_LOCATION = 'asia-south1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317'  // Replace with your project ID
        REPOSITORY = 'jenkins-cicd-stack'  // Your GAR repository name
        IMAGE_NAME = 'java-springboot'  // Change this if needed
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

        stage('List and Delete Old Image Tags') {
            steps {
                script {
                    def imageTags = sh(
                        script: """
                            gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME} --format='value(tag)'
                        """,
                        returnStdout: true
                    ).trim().split("\n")

                    if (imageTags.size() == 0) {
                        echo "No images found in ${IMAGE_NAME} repository."
                        return
                    }

                    for (tag in imageTags) {
                        if (tag != "latest") {
                            echo "Deleting image: ${IMAGE_NAME}:${tag}"
                            sh """
                                gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}@$(gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME} --filter="tags:${tag}" --format="value(DIGEST)") --quiet --delete-tags
                            """
                        }
                    }
                }
            }
        }
    }
}
