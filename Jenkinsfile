pipeline {
    agent any

    environment {
        REMOTE_HOST = "172.18.234.92"        // IP твоєї WSL
        REMOTE_USER = "nhoichuk"            // твій користувач WSL
        SSH_KEY_ID   = "jenkins-ssh-key-id" // ID ключа, який ти створила в Jenkins
        LOG_FILE     = "/var/log/apache2/access.log"
    }

    stages {
        stage('Start Apache') {
            steps {
                sshagent([env.SSH_KEY_ID]) {
                    sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} '
                        # Створюємо потрібні директорії
                        sudo mkdir -p /var/run/apache2 /var/log/apache2
                        sudo chown -R www-data:www-data /var/run/apache2 /var/log/apache2

                        # Встановлюємо змінні середовища
                        export APACHE_RUN_DIR=/var/run/apache2
                        export APACHE_LOG_DIR=/var/log/apache2
                        export APACHE_PID_FILE=/var/run/apache2/apache2.pid

                        # Встановлюємо Apache, якщо його нема
                        if ! command -v apache2 >/dev/null 2>&1; then
                            sudo apt update -y
                            sudo apt install apache2 -y
                        fi

                        # Запускаємо Apache
                        sudo /usr/sbin/apache2 -k start
                    '
                    """
                }
            }
        }

        stage('Check Apache Logs') {
            steps {
                sshagent([env.SSH_KEY_ID]) {
                    sh """
                    ssh ${REMOTE_USER}@${REMOTE_HOST} '
                        echo "4xx errors:"
                        sudo grep "HTTP/1.[01]\" [4][0-9][0-9]" ${LOG_FILE} || echo "No 4xx errors found"

                        echo "5xx errors:"
                        sudo grep "HTTP/1.[01]\" [5][0-9][0-9]" ${LOG_FILE} || echo "No 5xx errors found"
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}






