pipeline{
    agent { label 'jenkins-agent'}
    tools{
        jdk 'java17'
        maven 'maven'
    }
    environment{
        APP_NAME = "devops-project"
        RELEASE = "1.0.0"
        DOCKER_USER = "mukeshr29"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("jenkins-api")
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
        stage("Trivy scan"){
            steps{
                script{
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mukeshr29/devops-project:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage("cleanup artifact"){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage("memory clean up"){
            steps{
                script{
                    sh "docker system prune -y"
                }
            }
        }
        stage("trigger cd pipeline"){
            steps{
                script{
                    sh "curl -v -k --user mukesh:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://54.211.82.54:8080/job/gitops-project-7/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}