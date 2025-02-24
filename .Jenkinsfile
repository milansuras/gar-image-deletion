pipeline {
    agent any

    environment {
        HOST_NAME = 'asia-south1-docker.pkg.dev'
        PROJECT_ID = 'milan-dev-451317'
        REPOSITORY = 'jenkins-cicd-stack' // Ensure this matches the actual repository name
    }

    stages {
        stage('List and Display Image Tags') {
            steps {
                script {
                    sh '''
                    IMAGES=$(gcloud artifacts docker images list ${HOST_NAME}/${PROJECT_ID}/${REPOSITORY} \
                        --format="value(PACKAGE,DIGEST)")

                    echo "IMAGE_NAME | DIGEST | TAGS"

                    # Read each image and fetch tags
                    echo "$IMAGES" | while read -r IMAGE DIGEST; do
                        TAGS=$(gcloud artifacts docker tags list "${HOST_NAME}/${PROJECT_ID}/${IMAGE}" \
                            --format="value(TAG)" 2>/dev/null)

                        if [ -z "$TAGS" ]; then
                            TAGS="NO TAGS"
                        fi

                        echo -e "$IMAGE\t$DIGEST\t$TAGS"
                    done
                    '''
                }
            }
        }
    }
}
