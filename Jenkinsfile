pipeline{
    agent{
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "lfreez123"
        DOCKER_PASS = 'docker-hub-passwd'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
	
    stages{
        stage("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
    
        stage("Checkout from SCM"){
            steps {
                git branch: 'main', credentialsId: 'github-access-token', url: 'https://github.com/rasikaadl/project-1-application'
            }
	    }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
	    }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }
	    }

        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-access-token') {
                        sh "mvn sonar:sonar"
                    }
                }  
            } 
        } 

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-access-token' 
                }
            }
        }
	    
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }    
    }
}
