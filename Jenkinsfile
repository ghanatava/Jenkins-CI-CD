pipeline{
    agent any
	tools {
		maven 'maven'
	}
		stages{

			stage ("Code stability"){
				steps{
					sh "mvn clean install"
				}
			}

			stage ("code quality"){
				steps{
					sh "mvn checkstyle:checkstyle"
					recordIssues(tools: [checkStyle(pattern: '**/checkstyle-result.xml')])
				}
			}

			stage ("Unit testing") {
				steps{
					sh "mvn test"
					recordIssues(tools: [junitParser(pattern: 'target/surefire-reports/*.xml')])
				}
			}

			stage ("Security testing"){
				steps{
					sh "mvn org.owasp:dependency-check-maven:check"
					publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target', reportFiles: 'dependency-check-report.html',
					reportName: 'Dependency-check-report', reportTitles: ''])
				}
			}

			stage ("SonarQube Analysis"){
				steps{
					script{
						withCredentials([usernamePassword(credentialsId: 'sonar', usernameVariable: 'SONAR_USER',passwordVariable: 'SONAR_PASS')]){
							sh ''' 
								mvn sonar:sonar -Dsonar.host.url=http://sonarqube:9000  -Dsonar.login=${SONAR_USER} -Dsonar.password=${SONAR_PASS} -Dsonar.java.binaries=.
							'''
						}
					}
				}
			}

			stage('Deploy Artifact to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081/',
                        groupId: 'org',
                        version: 'v0.1',
                        repository: 'spring3hibernate',
                        credentialsId: 'nexus',
                        artifacts: [
                            [artifactId: 'spring3hibernate', classifier: '', file: 'target/Spring3HibernateApp.war', type: 'war']
                        ]
                    )
                }
            }
        }
	    }
}
