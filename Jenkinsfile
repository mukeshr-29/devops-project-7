pipeline{
    agent { label 'jenkins-agent'}
    tools{
        jdk 'java17'
        maven 'maven'
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
    }
}