pipeline {
    agent any

    stages {
        stage('Query Prisma Cloud API for Single Container Defender') {
            steps {
                script {
                    // Define API credentials and endpoint
                    def apiEndpoint = 'https://<prisma_cloud_api_endpoint>/defenders/docker-image-name'
                    def authToken = 'YOUR_AUTH_TOKEN' // Securely store and retrieve this

                    // API call to get Docker Image Name for Defender
                    def defenderDockerImageName = sh(script: """
                        curl -X GET "$apiEndpoint" \
                        -H "Authorization: Bearer $authToken"
                    """, returnStdout: true).trim()

                    // Process the response
                    println("Defender Docker Image Name: ${defenderDockerImageName}")
                }
            }
        }
    }
}
