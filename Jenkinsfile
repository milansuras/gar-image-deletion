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
                    
                    def tagsOutput = sh(
                        script: "gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}",
                        returnStdout: true
                    ).trim()
                    
                    echo "Tags output: ${tagsOutput}"
                    
                    
                    def lines = tagsOutput.split("\n")
                    env.LATEST_DIGEST = ""
                    
                    for (line in lines) {
                        if (line.contains("latest")) {
                            
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
                    
                    if (!env.LATEST_DIGEST) {
                        error "Failed to retrieve the digest for the 'latest' tag. Aborting to prevent accidental deletion of all images."
                    }
                }
            }
        }


        stage("Delete Non-Latest Images") {
            steps {
                script {
                    // Get all image digests
                    def allDigests = sh(
                        script: "gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --format='value(DIGEST)'",
                        returnStdout: true
                    ).trim().split("\n")
                    
                    echo "Found ${allDigests.size()} images in the repository"
                    
                    // Loop through all digests and delete those that don't match the latest
                    allDigests.each { digest ->
                        if (digest != env.LATEST_DIGEST) {
                            echo "Deleting image with digest: ${digest}"
                            sh "gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}@${digest} --quiet"
                        } else {
                            echo "Preserving latest image with digest: ${digest}"
                        }
                    }
                }
            }
        }
    }
}
