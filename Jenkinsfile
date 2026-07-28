pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Deploy') {
            steps {

                withCredentials([

                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.USER',
                           variable: 'ANYPOINT_CONNECTED_APP_USER'),

                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.PASSWORD',
                           variable: 'ANYPOINT_CONNECTED_APP_PASSWORD'),

                    string(credentialsId: 'ANYPOINT.USERNAME',
                           variable: 'ANYPOINT_USERNAME'),

                    string(credentialsId: 'ANYPOINT.PASSWORD',
                           variable: 'ANYPOINT_PASSWORD')

                ]) {

                    configFileProvider([
                        configFile(
                            fileId: 'Mulesoft Settings',
                            variable: 'MAVEN_SETTINGS'
                        )
                    ]) {

                        bat 'mvn -s "%MAVEN_SETTINGS%" deploy -DmuleDeploy'

                    }
                }
            }
        }

    }

}