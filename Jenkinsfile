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
                    withSonarQubeEnv('SonarQube') { 
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
    } 
}
