pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }


        stage('Sync Deployments with rsync') {
            steps {
                script {
                    withCredentials([ 
                        string(credentialsId: 'HOST_IP', variable: 'SERVER_HOST'),
                        string(credentialsId: 'SERVER_USER', variable: 'SERVER_USER'),
                        file(credentialsId: 'SERVER_KEY', variable: 'SSH_KEY_PATH'),
                        string(credentialsId: 'SERVER_PORT', variable: 'SSH_PORT')
                    ]) 
                    {
                        def path = '/var/www/html/pujan'
                        sh """
                            scp -i \$SSH_KEY_PATH -P \$SSH_PORT -r -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null * \${SERVER_USER}@\${SERVER_HOST}:${path} && ssh -i \$SSH_KEY_PATH -p \$SSH_PORT -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \${SERVER_USER}@\${SERVER_HOST} "sudo systemctl reload nginx"
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            emailext (
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Success",
                body: """<p>The build was successful.</p>
                <p>Build URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                <p>Branch: ${env.BRANCH_NAME}</p>""",
                recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider']]
            )
        }
        failure {
            emailext (
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Failed",
                body: """<p>The build failed.</p>
                <p>Build URL: <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>
                <p>Branch: ${env.BRANCH_NAME}</p>
                <p>Please check the console output for more details.</p>""",
                recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider']]
            )
        }
        always {
            cleanWs()
        }
    }
}
