pipeline {
    agent any
    parameters {
        string(name: 'annotations', description: 'Key-value pairs of annotations metadata.')
        booleanParam(name: 'bottlerocket', defaultValue: true, description: 'Deploy on Bottlerocket Linux OS.')
        string(name: 'cluster', description: 'Kubernetes or ECS cluster name.')
        booleanParam(name: 'collectPodLabels', defaultValue: true, description: 'Collect pod related labels.')
        string(name: 'consoleAddr', description: 'Console address for defender communication.')
        choice(name: 'containerRuntime', choices: ['docker', 'containerd', 'crio'], description: 'Container runtime type.')
        int(name: 'cpuLimit', defaultValue: 0, description: 'CPU limit for the defender daemonset.')
        string(name: 'credentialID', description: 'Name of the credential used.')
        string(name: 'dockerSocketPath', description: 'Path of the docker socket file.')
        booleanParam(name: 'gkeAutopilot', defaultValue: true, description: 'Deployment for GKE Autopilot.')
        string(name: 'image', description: 'Full daemonset image name.')
        booleanParam(name: 'istio', defaultValue: true, description: 'Monitor Istio.')
        int(name: 'memoryLimit', defaultValue: 0, description: 'Memory limit for the defender daemonset.')
        string(name: 'namespace', description: 'Target daemonset namespaces.')
        string(name: 'nodeSelector', description: 'Key/value node selector.')
        string(name: 'orchestration', description: 'Orchestration type.')
        string(name: 'priorityClassName', description: 'Priority class name for the defender.')
        booleanParam(name: 'privileged', defaultValue: true, description: 'Run defenders as privileged.')
        string(name: 'projectID', description: 'Kubernetes cluster project ID.')
        // object 'proxy' can be defined as a json string
        string(name: 'proxy', description: 'JSON string for defender proxy configuration.')
        string(name: 'region', description: 'Kubernetes cluster location region.')
        string(name: 'roleARN', description: 'Role ARN to associate with the service account.')
        string(name: 'secretsname', description: 'Name of the secret to pull.')
        booleanParam(name: 'selinux', defaultValue: true, description: 'SELinux enforced on the target host.')
        booleanParam(name: 'serviceaccounts', defaultValue: true, description: 'Monitor service accounts.')
        booleanParam(name: 'talos', defaultValue: true, description: 'Deployment on Talos Linux k8s cluster.')
        string(name: 'tolerations', defaultValue: '', description: 'JSON string representing a list of tolerations for the defender daemonset.')
        booleanParam(name: 'uniqueHostname', defaultValue: true, description: 'Assign unique hostnames.')
    }
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
                archiveArtifacts artifacts: 'twistlock_daemonset_defender_helm.tar.gz', onlyIfSuccessful: true
                archiveArtifacts artifacts: 'twistlock_daemonset_defender.yaml', onlyIfSuccessful: true
            }
        }
    }
}
