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
                        -X GET -o 'twistlock_defender.tar.gz' \\
                       https://app0.cloud.twistlock.com/panw-app0-310/api/v1/images/twistlock_defender.tar.gz 
                    """
                }
            }
        }
        stage('Query Prisma Cloud API for Daemonset Defender helm chart') {
            steps {
                script {
                    // Use the PRISMA_TOKEN environment variable for authentication
                    sh """
                        curl -k \\
                        -H 'Authorization: Bearer ${PRISMA_TOKEN}' \\
                        -H 'Content-Type: text/csv' \\
                        -X GET -o 'twistlock_daemonset_defender_helm.tar.gz' \\
                       https://app0.cloud.twistlock.com/panw-app0-310/api/v1/defenders/helm/twistlock-defender-helm.tar.gz 
                    """
                }
            }
        }
        stage('Query Prisma Cloud API for Daemonset Defender yaml') {
            steps {
                script {
                    // Use the PRISMA_TOKEN environment variable for authentication
                    sh """
                        curl -k \\
                        -H 'Authorization: Bearer ${PRISMA_TOKEN}' \\
                        -H 'Content-Type: text/csv' \\
                        -X GET -o 'twistlock_daemonset_defender.yaml' \\
                       https://app0.cloud.twistlock.com/panw-app0-310/api/v1/defenders/daemonset.yaml 
                    """
                }
            }
        }
        stage('Create Artifacts') {
            steps {
                archiveArtifacts artifacts: 'twistlock_defender.tar.gz', onlyIfSuccessful: true
            }
            steps {
                archiveArtifacts artifacts: 'twistlock_daemonset_defender_helm.tar.gz', onlyIfSuccessful: true
            }
            steps {
                archiveArtifacts artifacts: 'twistlock_daemonset_defender.yaml', onlyIfSuccessful: true
            }
        }
    }
}
