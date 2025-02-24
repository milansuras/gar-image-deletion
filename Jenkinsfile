pipeline {
    agent any

    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGE        = "java-backend"
        SERVICE_ACCOUNT = "jenkins-sa@milan-dev-451317.iam.gserviceaccount.com"
    }

    stages {
        stage("Grant IAM Permissions") {
            steps {
                sh """
                echo "Granting Artifact Registry permissions to Jenkins service account..."
                gcloud projects add-iam-policy-binding ${PROJECT_ID} \\
                    --member="serviceAccount:${SERVICE_ACCOUNT}" \\
                    --role="roles/artifactregistry.admin"
                """
            }
        }

        stage("Image Deletion") {
            steps {
                sh """
                set -e  # Exit on error

                # Fetch all image digests
                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --format="value(DIGEST)" > digest.txt

                # Ensure digest.txt is not empty
                if [ ! -s digest.txt ]; then
                    echo "No images found. Exiting."
                    exit 0
                fi

                # Process each digest
                rm -f digest_list.txt
                while read digest; do
                    tags=\$(gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --filter="digest=\${digest}" --format="value(tag)")
                    if ! echo "\$tags" | grep -q 'latest'; then 
                        echo "\$digest" >> digest_list.txt
                    fi
                done < digest.txt

                # Ensure digest_list.txt is not empty before deletion
                if [ ! -s digest_list.txt ]; then
                    echo "No old images to delete."
                    exit 0
                fi

                # Delete old images
                cat digest_list.txt | xargs -I {} gcloud artifacts docker images delete "${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}@{}" --delete-tags --quiet
                """
            }
        }
    }
}
