pipeline { 
    agent any

    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGE        = "python-backend"
    }

    stages {
        stage("List Images and Tags") {
            steps {
                sh """
                echo "Listing all images and tags:"
                gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}
                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}
                """
            }
        }
        
        stage("Identify Latest Image") {
            steps {
                script {
                    // Get the full output of tags to parse
                    def tagsOutput = sh(
                        script: "gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}",
                        returnStdout: true
                    ).trim()
                    
                    echo "Tags output: ${tagsOutput}"
                    
                    // Initialize variables properly
                    env.LATEST_DIGEST = ""
                    def nonLatestTags = []
                    
                    // Parse the output to extract information about all tags
                    def lines = tagsOutput.split("\n")
                    // Skip header lines (typically first two lines)
                    for (int i = 2; i < lines.size(); i++) {
                        def line = lines[i].trim()
                        if (line) {
                            def parts = line.split("\\s+")
                            if (parts.length >= 3) {
                                def tag = parts[0]
                                def digest = parts[2]
                                
                                if (tag == "latest") {
                                    env.LATEST_DIGEST = digest
                                    echo "Found latest tag with digest: ${digest}"
                                } else {
                                    // Store non-latest tags
                                    nonLatestTags.add(tag)
                                    echo "Found non-latest tag: ${tag} with digest: ${digest}"
                                }
                            }
                        }
                    }
                    
                    // Store non-latest tags as a comma-separated string
                    env.NON_LATEST_TAGS = nonLatestTags.join(',')
                    
                    echo "Latest image digest: ${env.LATEST_DIGEST}"
                    echo "Non-latest tags: ${env.NON_LATEST_TAGS}"
                    
                    // Check if we got a valid digest
                    if (!env.LATEST_DIGEST) {
                        error "Failed to identify the 'latest' tag. Aborting to prevent accidental deletion of all images."
                    }
                }
            }
        }
        
        stage("Remove Non-Latest Tags") {
            steps {
                script {
                    // If there are non-latest tags, remove them
                    if (env.NON_LATEST_TAGS) {
                        def tagsList = env.NON_LATEST_TAGS.split(',')
                        tagsList.each { tag ->
                            echo "Removing tag: ${tag}"
                            sh "gcloud artifacts docker tags delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}:${tag} --quiet"
                        }
                    } else {
                        echo "No non-latest tags to remove"
                    }
                }
            }
        }
        
        stage("Delete Untagged Images") {
            steps {
                script {
                    // Now get all images again - some may be untagged now
                    def imagesOutput = sh(
                        script: "gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}",
                        returnStdout: true
                    ).trim()
                    
                    echo "Images after tag removal: ${imagesOutput}"
                    
                    // Parse to get all digests
                    def lines = imagesOutput.split("\n")
                    
                    // Skip header lines (typically first two lines)
                    for (int i = 2; i < lines.size(); i++) {
                        def line = lines[i].trim()
                        if (line) {
                            def parts = line.split("\\s+")
                            if (parts.length >= 2) {
                                def digest = parts[1]
                                
                                if (digest != env.LATEST_DIGEST) {
                                    echo "Attempting to delete image with digest: ${digest}"
                                    try {
                                        sh "gcloud artifacts docker images delete ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}@${digest} --quiet"
                                        echo "Successfully deleted image with digest: ${digest}"
                                    } catch (Exception e) {
                                        echo "Warning: Failed to delete image with digest: ${digest}. It might still have tags or be referenced by other images."
                                    }
                                } else {
                                    echo "Preserving latest image with digest: ${digest}"
                                }
                            }
                        }
                    }
                }
            }
        }
        
        stage("Verify Results") {
            steps {
                sh """
                echo "Verifying remaining images and tags:"
                gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}
                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}
                """
            }
        }
    }
}