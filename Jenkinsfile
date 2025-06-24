pipeline {
    agent any

    environment {
        QA_EMAIL = 'srengty@gmail.com'
        ANSIBLE_PLAYBOOK = 'deploy_and_backup_laravel.yaml'
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                  composer install --no-interaction --prefer-dist --optimize-autoloader
                  php artisan test
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh "ansible-playbook ${ANSIBLE_PLAYBOOK}"
            }
        }
    }

    post {
        failure {
            script {
                def committers = sh(
                    script: "git log -1 --pretty=format:'%ae'",
                    returnStdout: true
                ).trim()

                emailext (
                    subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: "Build failed for ${env.JOB_NAME} #${env.BUILD_NUMBER}. Check Jenkins console output for details.",
                    to: "${QA_EMAIL}, ${committers}"
                )
            }
        }
    }
}
