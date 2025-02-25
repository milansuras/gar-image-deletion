pipeline { 
    agent any

    environment {
        GAR_LOCATION = "asia-south1-docker.pkg.dev"
        PROJECT_ID   = "milan-dev-451317"
        REPOSITORY   = "jenkins-cicd-satck"
        IMAGES       = "python-backend,java-backend,react-frontend"
    }

    stages {
        stage("Process All Repositories") {
            steps {
                script {
                    def imagesList = env.IMAGES.split(',')
                    
                    // Loop through each image repository
                    for (def image in imagesList) {
                        echo "=== Processing repository: ${image} ==="
                        processRepository(image)
                    }
                }
            }
        }
    }
}

// Function to process a single repository
def processRepository(String imageName) {
    stage("List Images for ${imageName}") {
        sh """
        echo "Listing all images and tags for ${imageName}:"
        gcloud artifacts docker images list ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName} --include-tags
        """
    }
    
    stage("Get Tag Information for ${imageName}") {
        script {
            // First, capture the output of the tags list command
            def tagsOutput = sh(
                script: "gcloud artifacts docker images list ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName} --include-tags",
                returnStdout: true
            ).trim()
            
            echo "Raw tags output for ${imageName}: ${tagsOutput}"
            
            // Write the output to a temporary file for easier processing
            def outputFile = "${imageName}_tags_output.txt"
            writeFile file: outputFile, text: tagsOutput
            
            // Find the latest tag digest using grep
            def latestTagLine = sh(
                script: "grep -w 'latest' ${outputFile} || echo ''",
                returnStdout: true
            ).trim()
            
            echo "Latest tag line for ${imageName}: ${latestTagLine}"
            
            if (latestTagLine) {
                // Extract the digest from the line containing "latest"
                def latestDigest = sh(
                    script: "echo '${latestTagLine}' | awk '{print \$2}'",
                    returnStdout: true
                ).trim()
                
                env["${imageName.toUpperCase().replaceAll('-', '_')}_LATEST_DIGEST"] = latestDigest
                echo "Latest digest for ${imageName}: ${latestDigest}"
            } else {
                echo "Warning: Could not find a 'latest' tag in repository ${imageName}. Skipping cleanup."
                return
            }
            
            // Find all non-latest tags
            def nonLatestTags = sh(
                script: "grep -v -w 'latest' ${outputFile} | grep -v '^TAG' | grep -v '^IMAGE' | grep -v '^-' | awk '{print \$3}'",
                returnStdout: true
            ).trim()
            
            if (nonLatestTags) {
                env["${imageName.toUpperCase().replaceAll('-', '_')}_NON_LATEST_TAGS"] = nonLatestTags.replaceAll("\\s+", ",")
                echo "Non-latest tags for ${imageName}: ${env["${imageName.toUpperCase().replaceAll('-', '_')}_NON_LATEST_TAGS"]}"
            } else {
                env["${imageName.toUpperCase().replaceAll('-', '_')}_NON_LATEST_TAGS"] = ""
                echo "No non-latest tags found for ${imageName}."
            }
        }
    }
    
    stage("Remove Non-Latest Tags for ${imageName}") {
        script {
            def nonLatestTagsVar = "${imageName.toUpperCase().replaceAll('-', '_')}_NON_LATEST_TAGS"
            
            if (env[nonLatestTagsVar] && env[nonLatestTagsVar] != "") {
                def tagsList = env[nonLatestTagsVar].split(",")
                for (tag in tagsList) {
                    if (tag && tag != "TAGS" && tag != "TAG") {  
                        echo "Removing tag: ${tag} from ${imageName}"
                        sh "gcloud artifacts docker tags delete ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName}:${tag} --quiet || echo 'Failed to delete tag ${tag}'"
                    } else if (tag == "TAGS" || tag == "TAG") {
                        echo "Skipping header value: ${tag}"
                    }
                }
            } else {
                echo "No non-latest tags to remove for ${imageName}."
            }
        }
    }
    
    stage("Delete Untagged Images for ${imageName}") {
        script {
            def latestDigestVar = "${imageName.toUpperCase().replaceAll('-', '_')}_LATEST_DIGEST"
            def latestDigest = env[latestDigestVar]
            
            if (!latestDigest) {
                echo "No latest digest defined for ${imageName}. Skipping untagged image cleanup."
                return
            }
            
            // Get list of all image digests
            def allDigests = sh(
                script: "gcloud artifacts docker images list ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName} --include-tags | grep -v '^IMAGE' | grep -v '^-' | awk '{print \$2}'",
                returnStdout: true
            ).trim()
            
            if (allDigests) {
                def digestsList = allDigests.split("\n")
                
                for (digest in digestsList) {
                    if (digest && digest != latestDigest) {
                        echo "Deleting image: ${digest} from ${imageName}"
                        try {
                            sh "gcloud artifacts docker images delete ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName}@${digest} --quiet || echo 'Failed to delete image ${digest}'"
                        } catch (Exception e) {
                            echo "Warning: Could not delete image ${digest} from ${imageName}. It might have tags or be referenced by another image. Error: ${e.message}"
                        }
                    } else {
                        echo "Preserving latest image: ${digest} for ${imageName}"
                    }
                }
            } else {
                echo "No images found to delete for ${imageName}."
            }
        }
    }
    
    stage("Verify Results for ${imageName}") {
        sh """
        echo "Verifying remaining images and tags for ${imageName}:"
        gcloud artifacts docker images list ${env.GAR_LOCATION}/${env.PROJECT_ID}/${env.REPOSITORY}/${imageName} --include-tags
        """
    }
}