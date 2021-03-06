pipeline {
    agent { label 'java' }
    environment {
        SITE_DIRECTORY = 'build/docs/html5'
    }
    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag')
    }
    stages {
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }
        stage('Demo') {
            steps {
                publishHTML([reportName  : 'Allure Docs', reportDir: env.SITE_DIRECTORY, reportFiles: 'index.html',
                             reportTitles: '', allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false])
            }
        }
        stage('Deploy') {
            when { branch "master" }
            steps {
                withCredentials([usernamePassword(credentialsId: 'qameta-ci_docker',
                        usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh './gradlew publishDockerImage -PimageTag=${IMAGE_TAG}'
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
        failure {
            slackSend message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} failed (<${env.BUILD_URL}|Open>)",
                    color: 'danger', teamDomain: 'qameta', channel: 'allure', tokenCredentialId: 'allure-channel'
        }
    }
}