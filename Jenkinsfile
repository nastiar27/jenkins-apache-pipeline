pipeline {
    agent any

    environment {
        REMOTE_HOST = "192.168.1.100"      // IP або хост віддаленої машини
        REMOTE_USER = "ubuntu"             // користувач на ВМ
        SSH_KEY_PATH = "/var/jenkins_home/.ssh/id_ed25519"  // шлях до приватного ключа у контейнері
    }

    stages {

        stage('Install Apache') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${SSH_KEY_PATH} ${REMOTE_USER}@${REMOTE_HOST} '
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
                script {
                    sh """
                    ssh -i ${SSH_KEY_PATH} ${REMOTE_USER}@${REMOTE_HOST} 'sudo systemctl status apache2 | head -n 20'
                    """
                }
            }
        }

        stage('Check Logs for Errors') {
            steps {
                script {
                    sh """
                    ssh -i ${SSH_KEY_PATH} ${REMOTE_USER}@${REMOTE_HOST} '
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




