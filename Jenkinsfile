pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }


    parameters {
        choice(
            name: 'CONFIG_ENV',
            choices: ['dev', 'qa', 'prod'],
            description: 'Select Deployment Configuration'
        )
        
        choice(
	        name: 'ANYPOINT_ENV',
	        choices: ['Sandbox', 'Design', 'Production'],
	        description: 'Select Anypoint Environment'
	    )
    }


    environment {
        BASE_VERSION = "1.0"
    }


    stages {

        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Generate Version') {
            steps {
                script {

                    def revision = bat(
                        script: '@mvn help:evaluate -Dexpression=revision -q -DforceStdout',
                        returnStdout: true
                    ).trim().readLines().last()

                    if (!revision) {
                        error "Unable to determine revision from pom.xml"
                    }

                    def parts = revision.tokenize('.')

                    if (parts.size() < 2) {
                        error "Revision must be in Major.Minor.Patch format"
                    }

                    env.APP_VERSION = "${parts[0]}.${parts[1]}.${env.BUILD_NUMBER}"

                    echo ""
                    echo "========================================="
                    echo "Base Revision : ${revision}"
                    echo "Build Number  : ${env.BUILD_NUMBER}"
                    echo "Artifact Ver  : ${env.APP_VERSION}"
                    echo "Environment   : ${params.CONFIG_ENV}"
                    echo "========================================="
                    echo ""
                }
            }
        }


		stage('Validate Parameters') {
		
		    steps {
		
		        script {
		
		            def validCombinations = [
		                Sandbox   : ['dev', 'qa'],
		                Design    : ['dev', 'qa'],
		                Production: ['prod']
		            ]
		
		            if (!validCombinations[params.ANYPOINT_ENV].contains(params.ENV)) {
		
		                error """
		=====================================================
		
		Invalid Parameter Selection
		
		Application Properties : ${params.ENV}
		Anypoint Environment   : ${params.ANYPOINT_ENV}
		
		Allowed combinations:
		
		Sandbox    -> dev, qa
		Design     -> dev, qa
		Production -> prod
		
		=====================================================
		"""
		
		            }
		
		            echo """
		=====================================================
		Parameter Validation Successful
		
		Application Properties : ${params.ENV}
		Anypoint Environment   : ${params.ANYPOINT_ENV}
		
		=====================================================
		"""
		
		        }
		
		    }
		
		}
		


        stage('Build') {
            steps {

                bat """
                mvn clean package ^
                -Drevision=${env.APP_VERSION} ^
                -Denv=${params.CONFIG_ENV}
                """

                bat """
                mvn help:evaluate ^
                -Dexpression=project.version ^
                -q ^
                -DforceStdout ^
                -Drevision=${env.APP_VERSION}
                """

            }

            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }

        stage('Publish to Exchange') {

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

                        bat """
                        mvn deploy ^
                        -s "${env.MAVEN_SETTINGS}" ^
                        -DskipMuleDeploy=true ^
                        -Drevision=${env.APP_VERSION}
                        """
                    }

                }

            }

        }

        stage('Deploy to CloudHub 2.0') {

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

                        bat """
                        mvn mule:deploy ^
                        -s "${env.MAVEN_SETTINGS}" ^
                        -Drevision=${env.APP_VERSION} ^
                        -Denv=${params.CONFIG_ENV} ^
                        -Danypoint.environment=${params.ANYPOINT_ENV}
                        """

                    }

                }

            }

        }

    }



    post {

		success {
		    echo '''
		╔══════════════════════════════════════════════════════╗
		║                                                      ║
		║              M U L E S O F T   ✔                    ║
		║                                                      ║
		║         CloudHub 2.0 Deployment Successful           ║
		║                                                      ║
		╚══════════════════════════════════════════════════════╝
		'''
		    echo "Application : ch2-demo-api"
		    echo "Version     : ${env.APP_VERSION}"
		    echo "Environment : ${params.ENV}"
		    echo "Build No.   : ${env.BUILD_NUMBER}"
		    echo "Status      : SUCCESS"
		}

        failure {
            echo ""
            echo "========================================="
            echo "Deployment Failed"
            echo "Version     : ${env.APP_VERSION}"
            echo "Environment : ${params.ENV}"
            echo "========================================="
        }

        always {
            cleanWs()
        }

    }

}