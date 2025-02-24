pipeline{
    agent any


    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGE        =  "java-backend"
    }


    stages{
        stage("Image Deletion"){
            steps {
                sh """

                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --format="value(DIGEST)" > digest.txt

                cat digest.txt

                while read digest; do
                    tags=\$(gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --filter="digest=\${digest}"  --format="value(tag)")
                    if ! echo "\$tags" | grep -q 'latest' ; then 
                        echo "\$digest" >> digest_list.txt
                    fi

                done < digest.txt

                cat digest_list.txt



                """
            }
        }
    }
}