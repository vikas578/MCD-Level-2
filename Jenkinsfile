	pipeline {
	
	    agent any
	    
	     options {
		    skipDefaultCheckout(true)
		}
	    
	    
	    parameters {
		    choice(
		        name: 'ENV',
		        choices: ['dev', 'qa', 'prod'],
		        description: 'Select properties file'
		    )
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
			
						def baseVersion = bat(
						    script: '@mvn help:evaluate -Dexpression=revision -q -DforceStdout',
						    returnStdout: true
						).trim().readLines().last()
			
			            if (!baseVersion || baseVersion.contains("null")) {
			                error "revision is not defined correctly in pom.xml"
			            }
			
			            def parts = baseVersion.tokenize('.')
			            
			            if (parts.size() < 2) {
						    error "Invalid revision format in pom.xml. Expected format: Major.Minor.Patch"
						}
			
			            env.APP_VERSION = "${parts[0]}.${parts[1]}.${env.BUILD_NUMBER}"
			
			            echo "==========================================="
			            echo "Base Version : ${baseVersion}"
			            echo "Build Number : ${env.BUILD_NUMBER}"
			            echo "App Version  : ${env.APP_VERSION}"
			            echo "==========================================="
			        }
			    }
			}
	
	
	        
	        
	        stage('Build') {
	            steps {
	                bat """
					mvn clean package ^
					-Drevision=${env.APP_VERSION} ^
					-Denv=${params.ENV}
					"""
	            }
	
	            post {
	                success {
	                    archiveArtifacts artifacts: 'target/*.jar'
	                }
	            }
	        }
	        
	        
	        
	
	        stage('Deploy to Cloudhub 2.0') {
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
	                    ]){
	
	                      	bat """
							mvn -s "${env.MAVEN_SETTINGS}" deploy ^
							-DmuleDeploy ^
							-Drevision=${env.APP_VERSION} ^
							-Denv=${params.ENV}
							"""
	                    }
	            	}
	            }
	        }
	
	    }   // <-- End of stages
	
	    post {
	
	        success {
	            echo "Deployment to ${params.ENV} completed successfully."
	        }
	
	        failure {
	            echo "Deployment to ${params.ENV} failed."
	        }
	
	        always {
	            cleanWs()
	        }
	
	    }
	
	}       // <-- End of pipeline