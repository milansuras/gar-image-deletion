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
                gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --include-list 
                gcloud artifacts docker tags list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE}
                """
            }
        }
        
        stage("Get Tag Information") {
            steps {
                script {
                    // First, capture the output of the tags list command
                    def tagsOutput = sh(
                        script: "gcloud artifacts docker images list ${GAR_LOCATION}/${PROJECT_ID}/${REPOSITORY}/${IMAGE} --include-list",
                        returnStdout: true
                    ).trim()
                    
                    echo "Raw tags output: ${tagsOutput}"
                    
                    // Write the output to a temporary file for easier processing
                    writeFile file: 'tags_output.txt', text: tagsOutput
                    
                    // Find the latest tag digest using grep
                    def latestTagLine = sh(
                        script: "grep -w 'latest' tags_output.txt || echo ''",
                        returnStdout: true
                    ).trim()
                    
                    echo "Latest tag line: ${latestTagLine}"
                    
                    if (latestTagLine) {
                        // Extract the digest from the line containing "latest"
                        def latestDigest = sh(
                            script: "echo '${latestTagLine}' | awk '{print \$3}'",
                            returnStdout: true
                        ).trim()
                        
                        env.LATEST_DIGEST = latestDigest
                        echo "Latest digest: ${env.LATEST_DIGEST}"
                    } else {
                        error "Could not find a 'latest' tag in the repository."
                    }
                    
                    // Find all non-latest tags
                    def nonLatestTags = sh(
                        script: "grep -v -w 'latest' tags_output.txt | grep -v '^TAG' | awk '{print \$1}'",
                        returnStdout: true
                    ).trim()
                    
                    if (nonLatestTags) {
                        env.NON_LATEST_TAGS = nonLatestTags.replaceAll("\\s+", ",")
                        echo "Non-latest tags: ${env.NON_LATEST_TAGS}"
                    } else {
                        env.NON_LATEST_TAGS = ""
                        echo "No non-latest tags found."
                    }
                }
            }
        }
        
        
        
        
    }
}
