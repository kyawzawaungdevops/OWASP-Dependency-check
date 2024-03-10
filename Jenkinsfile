@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'testingkyaw')
    }

    stages {
        stage('Git Checkout') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                git branch: "main",
                    credentialsId: 'your-credentials-id',
                    url: "https://github.com/kyawzawaungdevops/Complete-DevOps-CI-CD-pipeline-using-GitOps-with-ArgoCD.git"
            }
        }

        stage('Unit Test maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }

        stage('Integration Test maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }

        stage('Static code analysis: Sonarqube') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    statiCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnBuild()
               }
            }
        }
        stage('Jar file Push : Jfrog ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "jfrog",
                        usernameVariable: "USER",
                        passwordVariable: "PASS"
                    )]) {
                        sh "jfrog rt config --interactive=false --url='http://52.90.194.144:8082/artifactory' --user=$USER --password=$PASS --interactive=false"
                        sh "jfrog rt u '/var/lib/jenkins/workspace/"Spring Boot Application pipeline"/target/spring-petclinic-0.0.1-SNAPSHOT.jar' 'test' --recursive=true"
                    }
                }
            }
        }

        stage('Docker Image Build') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Scan: trivy ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Push : DockerHub ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Cleanup : DockerHub ') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }
    }
}
