pipeline {
    agent any

    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGE        = "java-backend"
    }


        stage("Image Deletion") {
            steps {
                sh """
                gcloud auth list
                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --format="value(DIGEST)" > digest.txt

               
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
