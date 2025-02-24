pipeline { 
    agent any

    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGE        = "python-backend"
    }

    stages {
        stage("Image Deletion") {
            steps {
                sh """
                gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} 
                gcloud artifacts docker tags   list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} 

                def latestDigest = sh(
                        script: "gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --format='value(DIGEST)' --filter=\"TAG=latest\"",
                        returnStdout: true
                    ).trim()

                echo "Latest image digest: ${latestDigest}"
               """
            }
        }
    }
}
