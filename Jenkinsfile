pipeline {

    agent any

    stages {

        stage('Build') {
            steps {
                bat 'mvn clean package'
                bat 'mvn deploy'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
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
                            fileId: 'mulesoft-settings',
                            variable: 'MAVEN_SETTINGS'
                        )
                    ]) {

                        bat 'mvn -s "%MAVEN_SETTINGS%" deploy -DmuleDeploy'

                    }
                }
            }
        }

    }   // <-- End of stages

    post {

        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }

        always {
            cleanWs()
        }

    }

}       // <-- End of pipeline