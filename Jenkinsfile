pipeline {
    agent { label 'Jenkins-Agent' }
    
    tools {
        jdk 'JAVA21'
        maven 'Maven3'
    }
    
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/biswarup12/register-app'
            }
        }
        
        stage('Build Application') {
            steps {
                // Added -DskipTests because tests are executed in the next stage
                sh "mvn clean package -DskipTests"
            }
        }
        
        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
        
        stage("SonarQube Analysis") {
            steps {
                script {
                    // Wrapped the sh command inside the withSonarQubeEnv block
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    // Wait 30 seconds before checking Quality Gate
                    sleep time: 30, unit: 'SECONDS'

                    // Wait up to 5 minutes for Quality Gate result
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate(
                            abortPipeline: false,
                            credentialsId: 'jenkins-sonarqube-token'
                        )

                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
