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

                

               """
            }
        }


         stage("Get Latest Image Digest") {
            steps {
                script {
                    // Alternative approach to get the digest for latest tag
                    // First, get all tags with their digests
                    def tagsOutput = sh(
                        script: "gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}",
                        returnStdout: true
                    ).trim()
                    
                    echo "Tags output: ${tagsOutput}"
                    
                    // Parse the output to extract the digest for the latest tag
                    def lines = tagsOutput.split("\n")
                    env.LATEST_DIGEST = ""
                    
                    // Skip the header lines and find the line with "latest"
                    for (line in lines) {
                        if (line.contains("latest")) {
                            // Extract the digest (assuming format like "latest image-path@sha256:digest")
                            def parts = line.trim().split("\\s+")
                            if (parts.length >= 3) {
                                def digestPart = parts[2]
                                if (digestPart.startsWith("sha256:")) {
                                    env.LATEST_DIGEST = digestPart
                                    break
                                }
                            }
                        }
                    }
                    
                    echo "Latest image digest: ${env.LATEST_DIGEST}"
                    
                    // Check if we got a valid digest
                    if (!env.LATEST_DIGEST) {
                        error "Failed to retrieve the digest for the 'latest' tag. Aborting to prevent accidental deletion of all images."
                    }
                }
            }
        }
    }
}
