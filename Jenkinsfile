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
        always {
            // Clean up workspace
            cleanWs()
        }
        
        success {
            echo 'Deployment completed successfully!'
        }
        
        failure {
            echo 'Deployment failed. Please check the logs.'
        }
    }
}
