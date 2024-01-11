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
                        def loginResponse = sh(script: '''
                            curl -s -X POST "https://app0.cloud.twistlock.com/panw-app0-310/api/v1/authenticate" \
                            -H 'accept: application/json; charset=UTF-8' \
                            -H 'content-type: application/json' \
                            -d '{"username": "'"$PC_IDENTITY"'", "password": "'"$PC_SECRET"'"}'
                        ''', returnStdout: true).trim()

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
                    // Use the PRISMA_TOKEN environment variable for authentication
                    sh """
                        curl -k \\
                        -H 'Authorization: Bearer ${PRISMA_TOKEN}' \\
                        -H 'Content-Type: text/csv' \\
                        -X GET -o 'defender.tar' \\
                        https://app0.cloud.twistlock.com/panw-app0-310/api/v1/defenders/download?latest=true
                    """
                }
            }
        }

        stage('Create Artifact') {
            steps {
                archiveArtifacts artifacts: 'defender.tar', onlyIfSuccessful: true
            }
        }
    }
}
