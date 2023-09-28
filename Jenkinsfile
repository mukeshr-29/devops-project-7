pipeline{
    agent { label 'jenkins-agent'}
    tools{
        jdk 'java17'
        maven 'maven'
    }
    environment{
        APP_NAME = "devops-project-7"
        RELEASE = "1.0.0"
        DOCKER_USER = "mukeshr29"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("cleanup workspace"){
            steps{
                cleanWs()
            }
        }
        stage("checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/mukeshr-29/devops-project-7.git'
            }
        }
        stage("build application"){
            steps{
                sh 'mvn clean package'
            }
        }
        stage("Test application"){
            steps{
                sh 'mvn test'
            }
        }
        stage("quality check using sonarqube"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube'){
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage("quality gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId:'sonarqube'
                }
            }
        }
        stage("build and push docker image"){
            steps{
                script{
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
    }
}