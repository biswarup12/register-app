pipeline {
    agent { label 'Jenkins-Agent' }
    
    tools {
        jdk 'JAVA21'
        maven 'Maven3'
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "biswarup1706"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
        stage("Build & PushDocker Image") {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/',DOCKER_PASS) {
                        def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        docker_image.push()
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image biswarup1706/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
    }
}
    
