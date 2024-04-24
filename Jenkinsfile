pipeline {
   agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "MVN3"
    }

    stages {
	    stage ('scm') {
		    steps {
			    // Get some code from a GitHub repository
                git credentialsId: 'github', url: 'git@github.com:SanitD/jenkins_test.git'
				}
			}
	    stage ('print stage') {
		    steps {
			    sh 'echo "new stage"'
		    }
	    }
        stage('Build') {
            steps {
               // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true -f api-gateway clean package"

                // To run Maven on a Windows agent, use
                //bat "mvn -Dmaven.test.failure.ignore=true -f api-gateway clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit stdioRetention: '', testResults: 'api-gateway/target/surefire-reports/*.xml'
                    archiveArtifacts 'api-gateway/target/*.jar'
                }
            }
        }
	 stage('sonar analysis') {
	       steps {
		       script {
			       scannerHome = tool 'sonar';
			       withSonarQubeEnv('sonar') {
						sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=jenkins_test -Dsonar.projectName=jenkins_test -Dsonar.projectVersion=1.0 -Dsonar.projectBaseDir=$WORKSPACE -Dsonar.sources=$WORKSPACE -Dsonar.java.binaries=$WORKSPACE"
						}
				}
			}
	   }
	    stage('sonar quality gate') {
		    steps {
			    script {
				    sleep(30)
				    qg = waitForQualityGate()
				    if (qg.status != 'OK') {
					    error "pipeline aborted due to quality gate failure: ${qg.status}"
				    }
			    }
		    }
	    }
    }
    post {
	success {
            emailext body: "Please check console aouput at $BUILD_URL for more information\n", to: "sanit295@gmail.com", subject: 'Jenkinstraining - $PROJECT_NAME build completed sucessfully - Build number is $BUILD_NUMBER - Build status is $BUILD_STATUS' 
        }  
    }
}
