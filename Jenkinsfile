pipeline {
    agent any

    stages {
        stage('Prisma Cloud Login') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'pc-service-account', usernameVariable: 'PC_IDENTITY', passwordVariable: 'PC_SECRET')
                ]) {
                    script {
                        // Perform login and capture the response
                        def loginResponse = sh(script: """
                            curl -s -X POST "https://api0.prismacloud.io/login" \
                            -H "accept: application/json; charset=UTF-8" \
                            -H "content-type: application/json" \
                            -d '{"username": "${PC_IDENTITY}", "password": "${PC_SECRET}"}'
                        """, returnStdout: true).trim()

                        // Parse the response and extract the token
                        def parsedResponse = readJSON text: loginResponse
                        if (parsedResponse.token) {
                            // Set the token as an environment variable for later use
                            env.PRISMA_TOKEN = parsedResponse.token
                        } else {
                            error "Failed to obtain PRISMA token"
                        }
                    }
                }
            }
        }
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