pipeline {

  agent any

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build and Deploy') {
      steps {

        withCredentials([
          string(credentialsId: 'ANYPOINT_CONNECTED_APP_USER',
            variable: 'ANYPOINT_CONNECTED_APP_USER'),

          string(credentialsId: 'ANYPOINT_CONNECTED_APP_PASSWORD',
            variable: 'ANYPOINT_CONNECTED_APP_PASSWORD'),

          string(credentialsId: 'ANYPOINT_USERNAME',
            variable: 'ANYPOINT_USERNAME'),

          string(credentialsId: 'ANYPOINT_PASSWORD',
            variable: 'ANYPOINT_PASSWORD'),

        ]) {

          configFileProvider([
            configFile(fileId: 'mulesoft-settings',
              variable: 'MAVEN_SETTINGS')
          ]) {

            bat ""
            "
            mvn - s "%MAVEN_SETTINGS%"
            clean deploy - DmuleDeploy ""
            "

          }
        }
      }
    }

  }