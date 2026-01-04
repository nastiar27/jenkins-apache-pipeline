pipeline {
    agent any

    environment {
        // IP або хост віддаленої машини
        REMOTE_HOST = "192.168.1.100" 
        REMOTE_USER = "ubuntu"   // ім'я користувача на ВМ
        SSH_KEY_ID = "jenkins-ssh-key" // ID SSH ключа, доданого у Jenkins
    }

    stages {

        stage('Install Apache') {
            steps {
                // sshagent використовує ключ для підключення
                sshagent([env.SSH_KEY_ID]) {
                    // Підключення до ВМ та установка Apache
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_HOST} '
                        sudo apt update && 
                        sudo apt install -y apache2 &&
                        sudo systemctl enable apache2 &&
                        sudo systemctl start apache2
                    '
                    """
                }
            }
        }

        stage('Check Apache Status') {
            steps {
                sshagent([env.SSH_KEY_ID]) {
                    sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} 'sudo systemctl status apache2 | head -n 20'
                    """
                }
            }
        }

        stage('Check Logs for Errors') {
            steps {
                sshagent([env.SSH_KEY_ID]) {
                    sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} '
                        echo "----Checking for 4xx and 5xx errors in Apache logs----" &&
                        grep " 4[0-9][0-9]\\| 5[0-9][0-9]" /var/log/apache2/access.log || echo "No errors found"
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
    }
}

