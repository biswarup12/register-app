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
                git branch: 'main', credentialsId: 'github', url: 'https://github.com'
            }
        }
        
        stage('Build Application') {
            steps {
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
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar"
                    }
                }
            }
        }

        // --- ADD THIS NEW STAGE HERE ---
        stage("Quality Gate Check") {
            steps {
                script {
                    // Timeout prevents the pipeline from hanging forever if the webhook fails
                    timeout(time: 10, unit: 'MINUTES') {
                        // abortPipeline: true will FAIL the build if the Quality Gate fails
                        def qg = waitForQualityGate(abortPipeline: true)
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
