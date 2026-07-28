pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'vikas578',
                    url: 'https://github.com/vikas578/MCD-Level-2.git'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }

    }

}