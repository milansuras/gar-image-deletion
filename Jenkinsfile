pipeline {
    agent any

    environment {
        GAR_LOCATION = 'asia-south1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317'  // Ensure this is correct
        REPOSITORY = 'jenkins-cicd-satck'  // Fix the typo if needed
        IMAGE_NAME = 'react-todo'  // Change this if needed
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
                            gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME} --format='value(tag)' || echo ""
                        """,
                        returnStdout: true
                    ).trim().split("\n")

                    if (imageTags.isEmpty() || imageTags[0] == "") {
                        echo "No images found in ${IMAGE_NAME} repository."
                        return
                    }

                    for (tag in imageTags) {
                        if (tag != "latest") {
                            echo "Deleting image: ${IMAGE_NAME}:${tag}"
                            sh """
                                DIGEST=$(gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME} --filter="tags=${tag}" --format="value(DIGEST)")
                                if [[ -n "$DIGEST" ]]; then
                                    gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}@$DIGEST --quiet --delete-tags
                                else
                                    echo "No digest found for tag ${tag}, skipping..."
                                fi
                            """
                        }
                    }
                }
            }
        }
    }
}
