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
                gcloud artifacts docker image list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} 

               
            }
        }
    }
}
